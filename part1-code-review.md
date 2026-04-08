\# Part 1: Code Review \& Debugging



\## 1.1 Issues Identified



| Issue | Description | Impact |

|------|------------|--------|

| No input validation | Direct access to request data without checks | API crashes (500 errors) |

| No transaction handling | Two separate commits for product and inventory | Data inconsistency if one fails |

| SKU uniqueness missing | No check for duplicate SKUs | Duplicate products and business errors |

| Incorrect data modeling | Product stores `warehouse\_id` | Cannot support multiple warehouses |

| No error handling | No try-catch or rollback | Application crashes and poor debugging |

| Price not handled as decimal | Price taken directly from input | Precision issues in financial data |

| Inventory duplication risk | No unique constraint on inventory | Duplicate inventory records |

| Assumes all fields mandatory | No handling of optional fields | API failures for valid partial input |

| No quantity validation | `initial\_quantity` not validated | Negative or invalid stock values |

| No proper status codes | Always returns success response | Poor API design and client confusion |



\---



\## 1.2 Corrected Code



```python

from flask import request, jsonify

from sqlalchemy.exc import IntegrityError

from decimal import Decimal



@app.route('/api/products', methods=\['POST'])

def create\_product():

   data = request.get\_json()



   try:

       # Validate required fields

       required\_fields = \['name', 'sku', 'price']

       for field in required\_fields:

           if field not in data:

               return jsonify({"error": f"{field} is required"}), 400



       # Check SKU uniqueness

       existing\_product = Product.query.filter\_by(sku=data\['sku']).first()

       if existing\_product:

           return jsonify({"error": "SKU already exists"}), 400



       # Convert price safely

       try:

           price = Decimal(str(data\['price']))

       except:

           return jsonify({"error": "Invalid price format"}), 400



       # Create product (transaction start)

       product = Product(

           name=data\['name'],

           sku=data\['sku'],

           price=price

       )

       db.session.add(product)

       db.session.flush()  # Get product.id before commit



       # Create inventory if provided

       if 'warehouse\_id' in data and 'initial\_quantity' in data:

           inventory = Inventory(

               product\_id=product.id,

               warehouse\_id=data\['warehouse\_id'],

               quantity=data\['initial\_quantity']

           )

           db.session.add(inventory)



       # Commit both operations together

       db.session.commit()



       return jsonify({

           "message": "Product created",

           "product\_id": product.id

       }), 201



   except IntegrityError:

       db.session.rollback()

       return jsonify({"error": "Database integrity error"}), 400



   except Exception as e:

       db.session.rollback()

       return jsonify({"error": str(e)}), 500

