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

&#x20;   data = request.get\_json()



&#x20;   try:

&#x20;       # Validate required fields

&#x20;       required\_fields = \['name', 'sku', 'price']

&#x20;       for field in required\_fields:

&#x20;           if field not in data:

&#x20;               return jsonify({"error": f"{field} is required"}), 400



&#x20;       # Check SKU uniqueness

&#x20;       existing\_product = Product.query.filter\_by(sku=data\['sku']).first()

&#x20;       if existing\_product:

&#x20;           return jsonify({"error": "SKU already exists"}), 400



&#x20;       # Convert price safely

&#x20;       try:

&#x20;           price = Decimal(str(data\['price']))

&#x20;       except:

&#x20;           return jsonify({"error": "Invalid price format"}), 400



&#x20;       # Create product (transaction start)

&#x20;       product = Product(

&#x20;           name=data\['name'],

&#x20;           sku=data\['sku'],

&#x20;           price=price

&#x20;       )

&#x20;       db.session.add(product)

&#x20;       db.session.flush()  # Get product.id before commit



&#x20;       # Create inventory if provided

&#x20;       if 'warehouse\_id' in data and 'initial\_quantity' in data:

&#x20;           inventory = Inventory(

&#x20;               product\_id=product.id,

&#x20;               warehouse\_id=data\['warehouse\_id'],

&#x20;               quantity=data\['initial\_quantity']

&#x20;           )

&#x20;           db.session.add(inventory)



&#x20;       # Commit both operations together

&#x20;       db.session.commit()



&#x20;       return jsonify({

&#x20;           "message": "Product created",

&#x20;           "product\_id": product.id

&#x20;       }), 201



&#x20;   except IntegrityError:

&#x20;       db.session.rollback()

&#x20;       return jsonify({"error": "Database integrity error"}), 400



&#x20;   except Exception as e:

&#x20;       db.session.rollback()

&#x20;       return jsonify({"error": str(e)}), 500

