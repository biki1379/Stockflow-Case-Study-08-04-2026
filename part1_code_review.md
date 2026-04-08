# Part 1: Code Review & Debugging

## Issues Identified

1. No Input Validation  
The API directly accesses fields like `data['name']` without checking if they exist.  
This can cause runtime errors if any field is missing.

2. SKU Uniqueness Not Enforced  
There is no check to ensure SKU is unique across the system.  
This can lead to duplicate products.

3. No Transaction Management  
Two separate commits are used:
- One for product creation  
- One for inventory creation  

If the second operation fails, the product is still created → inconsistent data.

4. Incorrect Data Modeling  
Product is tied to a single warehouse using `warehouse_id`, which violates the requirement that products can exist in multiple warehouses.

5. Price Not Validated  
Price is directly used without ensuring it is a valid numeric value.

6. No Error Handling  
If any database operation fails, the system does not rollback or return meaningful errors.

7. Inventory Duplication Risk  
There is no constraint preventing duplicate inventory entries for the same product and warehouse.

8. Optional Fields Not Handled  
Fields like `initial_quantity` may not exist but are accessed directly.

---

## Impact in Production

- API crashes due to missing fields  
- Duplicate SKUs causing business inconsistencies  
- Partial data saved (product without inventory)  
- Incorrect warehouse-product relationship  
- Financial calculation issues due to invalid price  
- Poor user experience due to lack of proper error messages  

---

## Fixed Implementation

```python
from sqlalchemy.exc import IntegrityError
from flask import request, jsonify

@app.route('/api/products', methods=['POST'])
def create_product():
    data = request.json

    try:
        # Validate required fields
        required_fields = ['name', 'sku', 'price']
        for field in required_fields:
            if field not in data:
                return jsonify({"error": f"{field} is required"}), 400

        # Validate price
        try:
            price = float(data['price'])
        except ValueError:
            return jsonify({"error": "Invalid price"}), 400

        # Check SKU uniqueness
        existing = Product.query.filter_by(sku=data['sku']).first()
        if existing:
            return jsonify({"error": "SKU already exists"}), 400

        # Create product (independent of warehouse)
        product = Product(
            name=data['name'],
            sku=data['sku'],
            price=price
        )

        db.session.add(product)
        db.session.flush()

        # Create inventory if provided
        if 'warehouse_id' in data and 'initial_quantity' in data:
            inventory = Inventory(
                product_id=product.id,
                warehouse_id=data['warehouse_id'],
                quantity=data.get('initial_quantity', 0)
            )
            db.session.add(inventory)

        # Single transaction commit
        db.session.commit()

        return jsonify({
            "message": "Product created",
            "product_id": product.id
        }), 201

    except IntegrityError:
        db.session.rollback()
        return jsonify({"error": "Database integrity error"}), 400

    except Exception as e:
        db.session.rollback()
        return jsonify({"error": str(e)}), 500
