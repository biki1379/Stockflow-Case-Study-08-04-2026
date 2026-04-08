
# part2_database_design.md
md
# Part 2: Database Design

## Proposed Schema (SQL)
sql
COMPANY (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

WAREHOUSE (
    id INT PRIMARY KEY,
    company_id INT,
    name VARCHAR(255),
    FOREIGN KEY (company_id) REFERENCES COMPANY(id)
);

PRODUCT (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    sku VARCHAR(100) UNIQUE,
    price DECIMAL(10,2),
    is_bundle BOOLEAN DEFAULT FALSE
);

INVENTORY (
    id INT PRIMARY KEY,
    product_id INT,
    warehouse_id INT,
    quantity INT,
    UNIQUE(product_id, warehouse_id),
    FOREIGN KEY (product_id) REFERENCES PRODUCT(id),
    FOREIGN KEY (warehouse_id) REFERENCES WAREHOUSE(id)
);

SUPPLIER (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    contact_email VARCHAR(255)
);

PRODUCT_SUPPLIER (
    product_id INT,
    supplier_id INT,
    PRIMARY KEY(product_id, supplier_id)
);

INVENTORY_LOG (
    id INT PRIMARY KEY,
    product_id INT,
    warehouse_id INT,
    change INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

BUNDLE_ITEMS (
    bundle_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY(bundle_id, product_id)
);
