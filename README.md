# Web-Dev-III-assignment-lab-3
# Assignment 3 - Inventory and Data Management System

A beginner-friendly REST API built for the Web Dev III assignment. It uses only the allowed backend tools: **Node.js, Express.js, MongoDB, Mongoose and dotenv**.

## 1. What this project does

- Creates, reads, updates and deletes products (CRUD).
- Checks product details before saving them (Mongoose validation).
- Filters products by category, text and price.
- Sorts products and returns them page by page.
- Shows products that have reached their reorder level.
- Creates a category-wise inventory report using MongoDB aggregation.
- Restocks products or reduces stock safely without allowing negative stock.
- Uses one central error handler for API errors.

## 2. Project folders

```text
inventory-data-management-system/
|-- .env.example
|-- .gitignore
|-- package.json
|-- README.md
`-- src/
    |-- app.js
    |-- server.js
    |-- controllers/
    |   `-- productController.js
    |-- middleware/
    |   |-- asyncHandler.js
    |   |-- errorMiddleware.js
    |   `-- notFound.js
    |-- models/
    |   `-- Product.js
    `-- routes/
        `-- productRoutes.js
```

**What each important file means:**

- `Product.js`: defines the product fields and their validation rules.
- `productRoutes.js`: connects URL paths and HTTP methods to controller functions.
- `productController.js`: contains the actual work for each API endpoint.
- `app.js`: creates the Express app and adds middleware and routes.
- `server.js`: connects to MongoDB and starts the server.
- `errorMiddleware.js`: sends readable error messages when something goes wrong.
- `.env`: stores settings such as the MongoDB connection string and port. Do not upload this file publicly.

## 3. Requirements before running

Install:

1. Node.js 18 or later.
2. MongoDB running locally, or a MongoDB Atlas database.

Open a terminal in this project folder and run:

```bash
npm install
```

Create your own `.env` file by copying `.env.example`.

On Windows Command Prompt:

```cmd
copy .env.example .env
```

On macOS/Linux:

```bash
cp .env.example .env
```

Open `.env`. For a local MongoDB database, it should contain:

```env
MONGO_URI=mongodb://127.0.0.1:27017/inventory_db
PORT=5000
```

For MongoDB Atlas, replace the local connection string with your Atlas connection string. Keep your real password private.

Start the project:

```bash
npm start
```

When it works, the terminal should show that MongoDB is connected and the API is running at `http://localhost:5000`.

Open this address in a browser to check the server:

```text
http://localhost:5000/
```

## 4. Test the APIs in Postman

For every request, use `http://localhost:5000` as the base URL. For POST, PUT and PATCH requests, select **Body -> raw -> JSON**.

### A. Create a product

**Method:** `POST`  
**URL:** `http://localhost:5000/api/products`

Body:

```json
{
  "name": "Gel Pen Pack of 15",
  "sku": "STNY-PN-713",
  "category": "Stationery",
  "price": 120,
  "quantity": 40,
  "reorderLevel": 20,
  "supplier": "Cello Pens"
}
```

Expected status: **201 Created**. The response contains the new product and its MongoDB `_id`. Copy that `_id`; you will use it in the next requests.

### B. Get all products

**Method:** `GET`  
**URL:** `http://localhost:5000/api/products`

The response includes the total number of matching products, current page, total pages and the products.

### C. Filter, sort and paginate

Try this URL:

```text
http://localhost:5000/api/products?category=Stationery&search=pen&minPrice=50&maxPrice=500&sort=-price&page=1&limit=5
```

Query parameters:

| Parameter | Meaning | Example |
|---|---|---|
| `category` | Match a category | `Stationery` |
| `search` | Search name, SKU or supplier | `pen` |
| `minPrice` | Minimum price | `50` |
| `maxPrice` | Maximum price | `500` |
| `sort` | Sort field; add `-` for descending | `price` or `-price` |
| `page` | Page number | `1` |
| `limit` | Products per page (maximum 100) | `5` |

Allowed sort fields are `name`, `category`, `price`, `quantity` and `createdAt`. The default is newest products first.

### D. Get one product by ID

**Method:** `GET`  
**URL:** `http://localhost:5000/api/products/PASTE_PRODUCT_ID_HERE`

Replace `PASTE_PRODUCT_ID_HERE` with the `_id` from the create response.

### E. Update product details

**Method:** `PUT`  
**URL:** `http://localhost:5000/api/products/PASTE_PRODUCT_ID_HERE`

Body:

```json
{
  "name": "Gel Pen Pack of 15",
  "sku": "STNY-PN-713",
  "category": "Stationery",
  "price": 140,
  "quantity": 60,
  "reorderLevel": 20,
  "supplier": "Cello Pens"
}
```

Expected status: **200 OK**. You can send only the fields you want to change; the other fields stay the same.

### F. Restock or reduce stock

**Method:** `PATCH`  
**URL:** `http://localhost:5000/api/products/PASTE_PRODUCT_ID_HERE/stock`

Restock by 10:

```json
{
  "change": 10
}
```

Record a sale of 2:

```json
{
  "change": -2
}
```

A positive number adds stock. A negative number removes stock. The API rejects a sale if there is not enough stock, so quantity cannot become negative.

### G. Delete a product

**Method:** `DELETE`  
**URL:** `http://localhost:5000/api/products/PASTE_PRODUCT_ID_HERE`

Expected status: **200 OK** with a `Product deleted` message.

### H. Low-stock report

**Method:** `GET`  
**URL:** `http://localhost:5000/api/products/low-stock`

A product appears here when `quantity` is less than or equal to `reorderLevel`. To test it, create a product with quantity `5` and reorderLevel `10`.

### I. Category-wise inventory summary

**Method:** `GET`  
**URL:** `http://localhost:5000/api/products/summary`

The summary groups products by category and calculates:

- `totalItems`: number of product records in that category.
- `totalQuantity`: sum of all quantities in that category.
- `totalStockValue`: sum of price multiplied by quantity.
- `avgPrice`: average product price in that category.

## 5. Product fields in simple words

| Field | Meaning | Example |
|---|---|---|
| `name` | Product name | `Gel Pen Pack of 15` |
| `sku` | Unique product code | `STNY-PN-713` |
| `category` | Product group | `Stationery` |
| `price` | Price of one product | `120` |
| `quantity` | How many units are in stock | `40` |
| `reorderLevel` | Stock level at which we should reorder | `20` |
| `supplier` | Supplier or company name | `Cello Pens` |

The API also returns `stockValue`. This is a virtual field calculated as `price * quantity`. `createdAt` and `updatedAt` are added automatically by Mongoose.

## 6. API endpoint summary

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/products` | Create a product |
| GET | `/api/products` | List products, filter, sort and paginate |
| GET | `/api/products/:id` | Get one product |
| PUT | `/api/products/:id` | Update product details |
| PATCH | `/api/products/:id/stock` | Add or remove stock safely |
| DELETE | `/api/products/:id` | Delete a product |
| GET | `/api/products/low-stock` | Get low-stock products |
| GET | `/api/products/summary` | Get category-wise summary |

## 7. Concepts to remember for your practical/viva

- **Node.js:** runs JavaScript on the server.
- **Express.js:** helps create API routes such as GET, POST, PUT, PATCH and DELETE.
- **MongoDB:** stores product records as documents.
- **Mongoose:** connects Node.js to MongoDB and defines the product schema and validation.
- **CRUD:** Create, Read, Update and Delete.
- **Middleware:** a function that runs during the request/response process. Here it is used to parse JSON and handle errors.
- **Filtering:** showing only products that match a condition, such as category = Stationery.
- **Sorting:** arranging products, for example by price from high to low.
- **Pagination:** dividing a long list into smaller pages.
- **Aggregation:** asking MongoDB to calculate a report, such as total stock value per category.
- **dotenv:** reads settings from `.env`, so the database URL is not written directly in the source code.

## 8. Important note

This is the backend API for the assignment. It does not include a frontend or admin panel because the assignment restricts the project to Node.js, Express, Mongoose and dotenv. Use Postman to demonstrate the API requests and responses.
