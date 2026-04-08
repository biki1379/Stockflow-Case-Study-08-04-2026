# StockFlow Backend Case Study

## Overview
This repository contains my solution for the StockFlow Inventory Management case study.

---

## Part 1: Code Review & Debugging
- Identified issues like lack of validation, missing transactions, and SKU duplication risk
- Fixed using proper validation, transaction handling, and error handling

---

## Part 2: Database Design
- Designed normalized schema for products, warehouses, inventory, and suppliers
- Supported multi-warehouse inventory and bundle products

---

## Part 3: Low Stock Alerts API
- Implemented API to detect low-stock items based on recent sales
- Included supplier details for reordering
- Handled multiple warehouses and edge cases

---

## Assumptions
- Recent sales = last 30 days
- Default threshold used if not defined
- One primary supplier per product

---

## Tech Stack
- Python (Flask) / Java (conceptual)
- MySQL (schema design)
- REST API principles
