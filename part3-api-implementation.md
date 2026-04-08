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

   try:

       alerts = \[]



       # Step 1: Fetch product, inventory, warehouse, and supplier data

       results = db.session.execute("""

           SELECT 

               p.id AS product\_id,

               p.name AS product\_name,

               p.sku,

               p.threshold,

               w.id AS warehouse\_id,

               w.name AS warehouse\_name,

               i.quantity AS current\_stock,

               s.id AS supplier\_id,

               s.name AS supplier\_name,

               s.contact\_email

           FROM products p

           JOIN inventory i ON p.id = i.product\_id

           JOIN warehouses w ON i.warehouse\_id = w.id

           LEFT JOIN product\_suppliers ps ON p.id = ps.product\_id

           LEFT JOIN suppliers s ON ps.supplier\_id = s.id

           WHERE p.company\_id = :company\_id

       """, {"company\_id": company\_id})



       for row in results:



           # Step 2: Check if stock is below threshold

           if row.current\_stock >= row.threshold:

               continue



           # Step 3: Check recent sales (last 30 days)

           recent\_sales = db.session.execute("""

               SELECT COALESCE(SUM(quantity\_sold), 0)

               FROM sales

               WHERE product\_id = :pid

               AND created\_at >= NOW() - INTERVAL '30 days'

           """, {"pid": row.product\_id}).scalar()



           if recent\_sales == 0:

               continue



           # Step 4: Calculate average daily sales

           avg\_daily\_sales = recent\_sales / 30



           # Step 5: Calculate days until stockout

           if avg\_daily\_sales == 0:

               days\_until\_stockout = None

           else:

               days\_until\_stockout = int(row.current\_stock / avg\_daily\_sales)



           # Step 6: Build alert response

           alerts.append({

               "product\_id": row.product\_id,

               "product\_name": row.product\_name,

               "sku": row.sku,

               "warehouse\_id": row.warehouse\_id,

               "warehouse\_name": row.warehouse\_name,

               "current\_stock": row.current\_stock,

               "threshold": row.threshold,

               "days\_until\_stockout": days\_until\_stockout,

               "supplier": {

                   "id": row.supplier\_id,

                   "name": row.supplier\_name,

                   "contact\_email": row.contact\_email

               } if row.supplier\_id else None

           })



       return {

           "alerts": alerts,

           "total\_alerts": len(alerts)

       }, 200



   except Exception as e:

       return {"error": str(e)}, 500

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

