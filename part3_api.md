# part3_api.md
md
# Part 3: Low Stock Alerts API

## Endpoint

GET /api/companies/{company_id}/alerts/low-stock

## Assumptions

 Recent sales = last 30 days  
 Default low stock threshold = 10 if not defined  
 Each product has at least one supplier  



## Implementation (Flask)
python
from datetime import datetime, timedelta
from sqlalchemy.sql import func

@app.route('/api/companies/<int:company_id>/alerts/low-stock', methods=['GET'])
def low_stock_alerts(company_id):
    alerts = []

    warehouses = Warehouse.query.filter_by(company_id=company_id).all()

    for warehouse in warehouses:
        inventories = db.session.query(Inventory, Product)\
            .join(Product, Inventory.product_id == Product.id)\
            .filter(Inventory.warehouse_id == warehouse.id)\
            .all()

        for inv, product in inventories:

            # Get recent sales (last 30 days)
            recent_sales = db.session.query(func.sum(Sales.quantity))\
                .filter(
                    Sales.product_id == product.id,
                    Sales.created_at >= datetime.utcnow() - timedelta(days=30)
                ).scalar() or 0

            if recent_sales == 0:
                continue

            threshold = getattr(product, 'low_stock_threshold', 10)

            if inv.quantity < threshold:

                daily_usage = recent_sales / 30 if recent_sales else 1
                days_until_stockout = int(inv.quantity / daily_usage) if daily_usage else None

                supplier = db.session.query(Supplier)\
                    .join(ProductSupplier)\
                    .filter(ProductSupplier.product_id == product.id)\
                    .first()

                alerts.append({
                    "product_id": product.id,
                    "product_name": product.name,
                    "sku": product.sku,
                    "warehouse_id": warehouse.id,
                    "warehouse_name": warehouse.name,
                    "current_stock": inv.quantity,
                    "threshold": threshold,
                    "days_until_stockout": days_until_stockout,
                    "supplier": {
                        "id": supplier.id if supplier else None,
                        "name": supplier.name if supplier else None,
                        "contact_email": supplier.contact_email if supplier else None
                    }
                })

    return {
        "alerts": alerts,
        "total_alerts": len(alerts)
    }
