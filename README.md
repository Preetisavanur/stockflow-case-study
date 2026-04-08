\# StockFlow - Inventory Management System Case Study



\## Candidate

Preeti V Savanur



\---



\## Overview

This repository contains my solution for the \*\*StockFlow B2B Inventory Management System case study\*\*.



The solution is divided into three parts:

\- Part 1: Code Review \& Debugging

\- Part 2: Database Design

\- Part 3: API Implementation



\---



\## Tech Stack

\- Python (Flask)

\- SQL (PostgreSQL - assumed)

\- SQLAlchemy (ORM)



\---



\## Repository Structure

stockflow-case-study/

│

├── README.md

├── part1-code-review.md

├── part2-database-design.md

├── part3-api-implementation.md

│

└── code/

&#x20;   └── app.py



\---



\## Assumptions

\- SKU is globally unique across the platform  

\- Low-stock threshold is defined per product  

\- Recent sales activity is considered within the last 30 days  

\- A product can have multiple suppliers  

\- Supplier information may be optional  

\- Days until stockout is calculated using average daily sales  



\---



\## Key Features Implemented

\- Input validation and error handling in API  

\- Transaction-safe product creation  

\- Scalable database design for multi-warehouse inventory  

\- Inventory tracking with audit logs  

\- Low-stock alert system with supplier details  

\- Support for bundle products  



\---



\## How to Run (Optional Setup)



1\. Clone the repository:

git clone  https://github.com/Preetisavanur/stockflow-case-study

cd stockflow-case-study



2\. Install dependencies:

pip install -r requirements.txt



3\. Run the application:

python code/app.py



\---



\## Notes

This solution includes assumptions due to intentionally incomplete requirements.  

Design decisions were made to ensure scalability, data consistency, and real-world applicability.



\---



\## Author

Preeti V Savanur

