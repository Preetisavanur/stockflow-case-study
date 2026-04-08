\# Part 3: API Implementation



\## 3.1 Endpoint



GET /api/companies/{company\_id}/alerts/low-stock



\---



\## 3.2 Assumptions



\- Recent sales activity is defined as the last 30 days  

\- Low-stock threshold is stored per product  

\- A product may or may not have a supplier  

\- Sales data is available in a sales table  

\- Days until stockout is calculated using average daily sales  



\---



\## 3.3 Implementation



```python

@app.route('/api/companies/<int:company\_id>/alerts/low-stock', methods=\['GET'])

def get\_low\_stock\_alerts(company\_id):

&#x20;   try:

&#x20;       alerts = \[]



&#x20;       # Step 1: Fetch product, inventory, warehouse, and supplier data

&#x20;       results = db.session.execute("""

&#x20;           SELECT 

&#x20;               p.id AS product\_id,

&#x20;               p.name AS product\_name,

&#x20;               p.sku,

&#x20;               p.threshold,

&#x20;               w.id AS warehouse\_id,

&#x20;               w.name AS warehouse\_name,

&#x20;               i.quantity AS current\_stock,

&#x20;               s.id AS supplier\_id,

&#x20;               s.name AS supplier\_name,

&#x20;               s.contact\_email

&#x20;           FROM products p

&#x20;           JOIN inventory i ON p.id = i.product\_id

&#x20;           JOIN warehouses w ON i.warehouse\_id = w.id

&#x20;           LEFT JOIN product\_suppliers ps ON p.id = ps.product\_id

&#x20;           LEFT JOIN suppliers s ON ps.supplier\_id = s.id

&#x20;           WHERE p.company\_id = :company\_id

&#x20;       """, {"company\_id": company\_id})



&#x20;       for row in results:



&#x20;           # Step 2: Check if stock is below threshold

&#x20;           if row.current\_stock >= row.threshold:

&#x20;               continue



&#x20;           # Step 3: Check recent sales (last 30 days)

&#x20;           recent\_sales = db.session.execute("""

&#x20;               SELECT COALESCE(SUM(quantity\_sold), 0)

&#x20;               FROM sales

&#x20;               WHERE product\_id = :pid

&#x20;               AND created\_at >= NOW() - INTERVAL '30 days'

&#x20;           """, {"pid": row.product\_id}).scalar()



&#x20;           if recent\_sales == 0:

&#x20;               continue



&#x20;           # Step 4: Calculate average daily sales

&#x20;           avg\_daily\_sales = recent\_sales / 30



&#x20;           # Step 5: Calculate days until stockout

&#x20;           if avg\_daily\_sales == 0:

&#x20;               days\_until\_stockout = None

&#x20;           else:

&#x20;               days\_until\_stockout = int(row.current\_stock / avg\_daily\_sales)



&#x20;           # Step 6: Build alert response

&#x20;           alerts.append({

&#x20;               "product\_id": row.product\_id,

&#x20;               "product\_name": row.product\_name,

&#x20;               "sku": row.sku,

&#x20;               "warehouse\_id": row.warehouse\_id,

&#x20;               "warehouse\_name": row.warehouse\_name,

&#x20;               "current\_stock": row.current\_stock,

&#x20;               "threshold": row.threshold,

&#x20;               "days\_until\_stockout": days\_until\_stockout,

&#x20;               "supplier": {

&#x20;                   "id": row.supplier\_id,

&#x20;                   "name": row.supplier\_name,

&#x20;                   "contact\_email": row.contact\_email

&#x20;               } if row.supplier\_id else None

&#x20;           })



&#x20;       return {

&#x20;           "alerts": alerts,

&#x20;           "total\_alerts": len(alerts)

&#x20;       }, 200



&#x20;   except Exception as e:

&#x20;       return {"error": str(e)}, 500

3.4 Approach

Fetch product, inventory, warehouse, and supplier data for the company

Filter products where current stock is below threshold

Check if the product has recent sales activity

Calculate average daily sales

Estimate days until stockout

Build and return structured alert response





3.5 Edge Cases Handled

Products with no recent sales are ignored

Division by zero is handled safely

Products without suppliers return null supplier

Multiple warehouses are handled using inventory table

Empty result returns an empty alerts list

Database errors handled using try-catch





3.6 Possible Improvements

Add pagination for large datasets

Use caching (e.g., Redis) for faster responses

Precompute alerts using background jobs (cron)

Add rate limiting for API protection

Optimize queries using indexes and better joins

