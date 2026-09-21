# Engineering Lab 04 — Fullstack Engineering Lab

# ShopKart: Wishlist System ❤️

**Duration:** 2 Hours  
**Mode:** Individual Lab  
**Total Marks:** 100

---

## The Story

Customers can now log into ShopKart and browse the complete product catalogue.

But a new problem has appeared.

> **“I like this product… but I’m not ready to buy it yet.”**

Modern e-commerce applications allow users to save products for later. Your task is to build the **Wishlist System** for ShopKart.

This lab introduces an important backend concept:

> **Relationships between users and products.**

Instead of storing wishlist data only in React, the wishlist must be stored permanently in MongoDB and associated with the authenticated user.

The wishlist should survive:

- Page refreshes
- Browser restarts
- Logging out and logging back in
- Opening the application from another device

---

# Learning Objectives

By the end of this lab, you should be able to:

- Model relationships using MongoDB ObjectIds
- Reference one MongoDB document from another
- Use \`Schema.Types.ObjectId\`
- Use Mongoose \`populate()\`
- Build protected REST APIs
- Use authentication middleware with user-specific data
- Prevent duplicate relationships
- Render relational API data in React
- Implement loading, empty and error states
- Understand why persistent application data should live in the backend

---

# What You'll Build

The final ShopKart flow should look like:

\`\`\`text
Login
   ↓
Products
   ↓
♡ Add to Wishlist
   ↓
Wishlist Page
   ↓
Remove from Wishlist
\`\`\`

A user should only be able to see and modify **their own wishlist**.

---

# Important Rules

1. Wishlist APIs must be protected.
2. A user cannot access another user's wishlist.
3. A product can appear only once in a user's wishlist.
4. Products must exist before they can be added.
5. Wishlist data must be stored in MongoDB.
6. Do not hardcode wishlist data in React.
7. Do not store the complete product object inside the wishlist.
8. Use product references and \`populate()\` to retrieve product information.

---

# Part 1 — Backend

## Task 1 — Update User Schema (15 Marks)

Extend the existing User model with a wishlist field.

### Requirement

The wishlist should contain references to the Product documents.

Example:

\`\`\`js
wishlist: [
  ObjectId("productId1"),
  ObjectId("productId2")
]
\`\`\`

### Suggested Schema

\`\`\`js
wishlist: [
  {
    type: mongoose.Schema.Types.ObjectId,
    ref: "Product"
  }
]
\`\`\`

### Requirements

- Use \`Schema.Types.ObjectId\`
- Reference the \`Product\` model
- Default value should be an empty array
- Existing users should continue to work after the schema change

### Concept Check

Why are we storing ObjectIds instead of copying the entire product?

Because the Product collection remains the **source of truth** for product information.

---

# Task 2 — Add Product to Wishlist (15 Marks)

Create an API to add a product to the authenticated user's wishlist.

### Endpoint

\`\`\`http
POST /wishlist/:productId
\`\`\`

### Authentication

This route must be protected using the authentication middleware from Lab-02.

The authenticated user's ID should come from the server-side authentication mechanism.

Do **not** accept \`userId\` from the request body.

### Example Request

\`\`\`http
POST /wishlist/66d123abc456...
Authorization: Bearer <token>
\`\`\`

### Expected Behaviour

1. Verify the user is authenticated.
2. Verify the product exists.
3. Check whether the product is already in the wishlist.
4. Add it if it is not present.
5. Save the updated user document.

### Success Response

Return status \`200\` or \`201\`.

Example:

\`\`\`json
{
  "success": true,
  "message": "Product added to wishlist"
}
\`\`\`

### Failure Cases

| Situation | Status Code |
|---|---:|
| User not authenticated | 401 |
| Product ID invalid | 400 |
| Product not found | 404 |
| Product already in wishlist | 409 |

### Duplicate Example

If the same product is added twice:

\`\`\`json
{
  "success": false,
  "message": "Product already exists in wishlist"
}
\`\`\`

The database should still contain only one reference.

---

# Task 3 — Get Current User's Wishlist (20 Marks)

Create an API that returns the wishlist of the currently authenticated user.

### Endpoint

\`\`\`http
GET /wishlist
\`\`\`

### Authentication

Protected route.

### Important

You must return **only the current user's wishlist**.

Do not accept a user ID through:

- Query parameters
- Request body
- URL parameters

The server must identify the user using the authentication middleware.

### Use Mongoose populate()

Populate the Product references so that the frontend receives useful product information.

### Example Response

\`\`\`json
{
  "success": true,
  "count": 2,
  "wishlist": [
    {
      "_id": "66d123...",
      "name": "Mechanical Keyboard",
      "price": 2999,
      "category": "Electronics",
      "image": "https://example.com/keyboard.jpg",
      "stock": 10
    },
    {
      "_id": "66d456...",
      "name": "Wireless Headphones",
      "price": 4999,
      "category": "Electronics",
      "image": "https://example.com/headphones.jpg",
      "stock": 5
    }
  ]
}
\`\`\`

### Performance Hint

You do not need to return every field of the Product document.

Return only the fields required by the Wishlist UI.

---

# Task 4 — Remove Product from Wishlist (10 Marks)

Create an API to remove a product from the current user's wishlist.

### Endpoint

\`\`\`http
DELETE /wishlist/:productId
\`\`\`

### Authentication

Protected route.

### Expected Behaviour

The server should:

1. Identify the authenticated user.
2. Remove the specified product reference.
3. Save the updated user.
4. Return a success response.

### Success Response

\`\`\`json
{
  "success": true,
  "message": "Product removed from wishlist"
}
\`\`\`

### Failure Cases

| Situation | Status Code |
|---|---:|
| User not authenticated | 401 |
| Invalid product ID | 400 |
| Product not found in wishlist | 404 |

---

# Backend Routes Summary

| Method | Endpoint | Purpose |
|---|---|---|
| POST | \`/wishlist/:productId\` | Add product |
| GET | \`/wishlist\` | Get current user's wishlist |
| DELETE | \`/wishlist/:productId\` | Remove product |

---

# Suggested Backend Structure

Follow MVC architecture.

\`\`\`text
backend/
│
├── controllers/
│   ├── auth.controller.js
│   ├── product.controller.js
│   └── wishlist.controller.js
│
├── models/
│   ├── user.model.js
│   └── product.model.js
│
├── routes/
│   ├── auth.routes.js
│   ├── product.routes.js
│   └── wishlist.routes.js
│
├── middlewares/
│   └── auth.middleware.js
│
├── utils/
│
└── index.js
\`\`\`

---

# Part 2 — Frontend

## Task 5 — Add Wishlist Button to Product Card (10 Marks)

Update the Product Card created in Lab-03.

Add a button such as:

\`\`\`text
♡ Wishlist
\`\`\`

When clicked:

\`\`\`text
User clicks Wishlist
        ↓
POST /wishlist/:productId
        ↓
Backend updates MongoDB
        ↓
Show success feedback
\`\`\`

### Requirements

- Do not refresh the page.
- Use the existing authenticated session/token mechanism.
- Disable the button while the request is running.
- Handle API failures gracefully.

### Suggested UI

Before adding:

\`\`\`text
♡ Add to Wishlist
\`\`\`

After adding:

\`\`\`text
♥ Added to Wishlist
\`\`\`

The exact visual design is up to you.

---

# Task 6 — Create Wishlist Page (15 Marks)

Create a new frontend route:

\`\`\`text
/wishlist
\`\`\`

The page should fetch data from:

\`\`\`http
GET /wishlist
\`\`\`

### Each Wishlist Card should display:

- Product image
- Product name
- Price
- Category
- Stock status
- View Details button
- Remove from Wishlist button

### Example

\`\`\`text
┌───────────────────────────────┐
│                               │
│        PRODUCT IMAGE          │
│                               │
├───────────────────────────────┤
│ Mechanical Keyboard           │
│ Electronics                   │
│ ₹2,999                        │
│ 10 units left                 │
│                               │
│ View Details   Remove ♥       │
└───────────────────────────────┘
\`\`\`

Products must be rendered dynamically from the API response.

---

# Task 7 — Wishlist UI States (5 Marks)

Your Wishlist page must handle three important states.

## Loading State

While the API request is running:

\`\`\`text
Loading wishlist...
\`\`\`

Do not show a blank screen.

## Empty State

When the wishlist contains zero products:

\`\`\`text
Your wishlist is empty ❤️

Start saving products you love.
\`\`\`

Add a button to take the user back to the Product Catalog.

## Error State

If the API request fails:

\`\`\`text
Unable to load wishlist.

Please try again.
\`\`\`

---

# Task 8 — Navigation Integration (5 Marks)

Update the Navbar created in the previous labs.

Add a Wishlist link:

\`\`\`text
Home | Products | Wishlist | Logout
\`\`\`

The Wishlist page must be accessible through React Router.

---

# Part 3 — Authentication & Authorization

This lab is also testing whether you correctly understand the authentication system built in Lab-02.

## Requirement

A user should never be able to do this:

\`\`\`text
GET /wishlist/user/anotherUserId
\`\`\`

There should be no API that allows a client to simply provide another user's ID and retrieve their wishlist.

### Correct Flow

\`\`\`text
JWT
 ↓
Authentication Middleware
 ↓
Authenticated User
 ↓
User's Wishlist
\`\`\`

The backend decides whose wishlist is accessed.

---

# Data Relationship

Think of the data model as:

\`\`\`text
User
 │
 └── wishlist
       │
       ├── Product ID
       ├── Product ID
       └── Product ID
\`\`\`

The relationship is:

\`\`\`text
User  ─────── references ───────> Product
\`\`\`

The Product remains in the Product collection.

The User only stores references.

---

# Why Not Store the Entire Product?

Avoid this:

\`\`\`js
wishlist: [
  {
    name: "Keyboard",
    price: 2999,
    image: "...",
    category: "Electronics"
  }
]
\`\`\`

Instead use:

\`\`\`js
wishlist: [
  ObjectId("...")
]
\`\`\`

Because:

- Product details can change.
- Product information should have one source of truth.
- Duplicating product data creates consistency problems.
- MongoDB references allow us to fetch the related document when required.

---

# Part 4 — API Testing

Before connecting the APIs to React, test the backend using Postman.

## Test Case 1 — Add Product

\`\`\`http
POST /wishlist/:productId
\`\`\`

Expected:

\`\`\`text
Product added successfully
\`\`\`

## Test Case 2 — Add Same Product Again

Expected:

\`\`\`text
409 Conflict
\`\`\`

## Test Case 3 — Get Wishlist

\`\`\`http
GET /wishlist
\`\`\`

Expected:

\`\`\`text
Only the authenticated user's products
\`\`\`

## Test Case 4 — Remove Product

\`\`\`http
DELETE /wishlist/:productId
\`\`\`

Expected:

\`\`\`text
Product removed successfully
\`\`\`

## Test Case 5 — Request Without Authentication

Call any protected wishlist endpoint without a valid token.

Expected:

\`\`\`text
401 Unauthorized
\`\`\`

---

# Suggested Frontend Structure

\`\`\`text
src/
│
├── pages/
│   ├── Login.jsx
│   ├── Register.jsx
│   ├── Home.jsx
│   ├── Products.jsx
│   ├── ProductDetails.jsx
│   └── Wishlist.jsx
│
├── components/
│   ├── Navbar.jsx
│   ├── ProductCard.jsx
│   └── WishlistCard.jsx
│
├── services/
│   └── api.js
│
├── context/
│
├── App.jsx
└── main.jsx
\`\`\`

You may organize the files differently as long as separation of concerns is maintained.

---

# Acceptance Criteria

A submission is considered complete when:

- User schema contains a wishlist field.
- Wishlist stores Product ObjectId references.
- Product references use the correct Mongoose \`ref\`.
- Add Wishlist API works.
- Duplicate wishlist entries are prevented.
- Get Wishlist API works.
- Wishlist API uses \`populate()\`.
- Remove Wishlist API works.
- All wishlist APIs are protected.
- Backend uses the authenticated user instead of accepting \`userId\` from the client.
- Product cards contain a Wishlist action.
- Wishlist page exists.
- Wishlist data is fetched from the backend.
- Wishlist products are rendered dynamically.
- Remove functionality works.
- Loading state is implemented.
- Empty state is implemented.
- Error state is implemented.
- Navbar contains Wishlist navigation.
- APIs have been tested using Postman.
- Code follows a reasonable MVC structure.

---

# TA Evaluation Rubric

| Category | Marks |
|---|---:|
| User Schema + MongoDB Relationship | 15 |
| Add Wishlist API | 15 |
| Get Wishlist API + populate | 20 |
| Remove Wishlist API | 10 |
| Product Card Integration | 10 |
| Wishlist Page | 15 |
| Loading / Empty / Error States | 5 |
| Code Quality / Structure | 5 |
| Viva | 5 |
| **Total** | **100** |

---

# TA Viva Questions

Ask any 3–5 depending on the student's implementation.

1. Why are we storing ObjectIds instead of complete Product objects?
2. What does the \`ref\` option do in a Mongoose schema?
3. What problem does \`populate()\` solve?
4. What is the difference between embedding and referencing in MongoDB?
5. Why should the wishlist API be protected?
6. Why should the backend identify the user from the JWT instead of accepting a \`userId\` from the frontend?
7. How would duplicate wishlist items occur?
8. How can you prevent duplicate wishlist entries?
9. What happens if the product referenced in the wishlist is deleted?
10. Why should the frontend not be treated as the source of truth for persistent wishlist data?

---

# Bonus Challenge (+10 Marks)

## Implement Wishlist Toggle

Replace separate Add and Remove interactions with a single toggle.

### Behaviour

If the product is not in the wishlist:

\`\`\`text
♡ Add to Wishlist
\`\`\`

After adding:

\`\`\`text
♥ Remove from Wishlist
\`\`\`

### Bonus API

You may implement:

\`\`\`http
PATCH /wishlist/:productId/toggle
\`\`\`

The API should:

- Add the product if it is not present.
- Remove the product if it is already present.
- Return the updated wishlist state.

---

# Bonus Challenge 2 — Wishlist Count (+5)

Display the number of wishlisted products in the Navbar.

Example:

\`\`\`text
Wishlist (3)
\`\`\`

The count must come from the backend response.

Do not hardcode it.

---

# Submission Rules

- Continue from the existing ShopKart application.
- Do not create a separate project.
- Use MongoDB as the source of truth.
- Do not hardcode wishlist data in React.
- Use JWT authentication from Lab-02.
- Test all backend APIs with Postman.
- Use React Router for Wishlist navigation.
- Follow MVC architecture on the backend.
- Students may use AI tools for reference, but must understand and explain their implementation during the viva.

---

# What's Next?

ShopKart is now getting closer to a real e-commerce application:

\`\`\`text
Authentication ✅
       ↓
Product Discovery ✅
       ↓
Wishlist ❤️
       ↓
Shopping Cart 🛒
       ↓
Checkout 💳
       ↓
Orders 📦
\`\`\`

The next phase will introduce **client-side state management** for the Shopping Cart.

You will then be able to discuss an important engineering question:

> **When should application state live in the backend, and when should it live in the frontend?**
