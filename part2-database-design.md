\# Part 2: Database Design



\## 2.1 Schema Design



\### Companies

\- id (INT, Primary Key)

\- name (VARCHAR(255))

\- created\_at (TIMESTAMP)



Relationship:

\- One company → many warehouses

\- One company → many products



\---



\### Warehouses

\- id (INT, Primary Key)

\- company\_id (INT, Foreign Key → companies.id)

\- name (VARCHAR(255))

\- location (VARCHAR(255))



Relationship:

\- Each warehouse belongs to one company



\---



\### Products

\- id (INT, Primary Key)

\- company\_id (INT, Foreign Key → companies.id)

\- name (VARCHAR(255))

\- sku (VARCHAR(100), UNIQUE)

\- price (DECIMAL(10,2))

\- product\_type (VARCHAR(50))  -- normal / bundle

\- threshold (INT)

\- created\_at (TIMESTAMP)



Relationship:

\- One product → multiple warehouses (via inventory)



\---



\### Inventory

\- id (INT, Primary Key)

\- product\_id (INT, Foreign Key → products.id)

\- warehouse\_id (INT, Foreign Key → warehouses.id)

\- quantity (INT)



Constraints:

\- UNIQUE(product\_id, warehouse\_id)



Relationship:

\- Many-to-many between products and warehouses



\---



\### Inventory Logs

\- id (INT, Primary Key)

\- product\_id (INT)

\- warehouse\_id (INT)

\- change (INT)

\- created\_at (TIMESTAMP)



Purpose:

\- Track inventory changes (audit/history)



\---



\### Suppliers

\- id (INT, Primary Key)

\- name (VARCHAR(255))

\- contact\_email (VARCHAR(255))



\---



\### Product\_Suppliers

\- product\_id (INT, Foreign Key → products.id)

\- supplier\_id (INT, Foreign Key → suppliers.id)



Constraints:

\- PRIMARY KEY(product\_id, supplier\_id)



Relationship:

\- Many-to-many between products and suppliers



\---



\### Bundles

\- id (INT, Primary Key)

\- parent\_product\_id (INT)

\- child\_product\_id (INT)

\- quantity (INT)



Purpose:

\- Defines bundle composition



\---



\### Sales (Assumed for alerts)

\- id (INT, Primary Key)

\- product\_id (INT)

\- quantity\_sold (INT)

\- created\_at (TIMESTAMP)



Purpose:

\- Used for calculating recent sales and stockout prediction



\---



\## 2.2 Identify Gaps (Questions for Product Team)



\- Should SKU be globally unique or scoped per company?  

\- Can a product have multiple suppliers, and how do we select a default supplier?  

\- What defines "recent sales activity" (e.g., last 7 days or 30 days)?  

\- Is the low-stock threshold defined per product or per warehouse?  

\- Can bundles contain other bundles (nested bundles)?  

\- How should days\_until\_stockout be calculated?  

\- Should inventory updates be real-time or batch processed?  



\---



\## 2.3 Design Decisions



\- Used a separate inventory table to support products across multiple warehouses  

\- Added UNIQUE(product\_id, warehouse\_id) to prevent duplicate inventory records  

\- Used DECIMAL data type for price to ensure financial precision  

\- Enforced SKU uniqueness to maintain product identity  

\- Created many-to-many relationship (product\_suppliers) for flexibility  

\- Added inventory\_logs table to track stock changes for auditing  

\- Added indexing on SKU for faster lookup  

\- Added indexing on (product\_id, warehouse\_id) for efficient queries  

\- Used foreign key constraints to maintain referential integrity  

\- Designed bundles table to support composite products  

\- Structured schema to support multi-tenant (company-based) architecture  

\- Allowed optional fields to support flexible API usage  

