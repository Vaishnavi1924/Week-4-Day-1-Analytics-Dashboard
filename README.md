# Week-4-Day-1-Analytics-Dashboard
Vendor Dashboard ├── Total Products ├── Total Orders ├── Total Sales ├── Total Customers ├── Pending Orders ├── Processing Orders ├── Shipped Orders └── Delivered Orders


Step 1 — Create Analytics Controller

Create:

backend/controllers/analyticsController.js

Add:

const Order = require("../models/Order");
const Product = require("../models/Product");
const User = require("../models/User");

const getVendorAnalytics = async (req, res) => {
  try {
    if (req.user.role !== "vendor") {
      return res.status(403).json({
        message: "Only vendors can access analytics"
      });
    }

    const vendor = await User.findById(
      req.user.userId
    ).select("storeId");

    if (!vendor || !vendor.storeId) {
      return res.status(400).json({
        message: "Please create a store first"
      });
    }

    const storeId = vendor.storeId;

    // Total products
    const totalProducts =
      await Product.countDocuments({
        storeId
      });

    // All orders for this store
    const orders = await Order.find({
      storeId
    });

    // Total orders
    const totalOrders = orders.length;

    // Total sales from paid orders
    const totalSales = orders
      .filter(
        (order) =>
          order.paymentStatus === "paid"
      )
      .reduce(
        (total, order) =>
          total + order.totalAmount,
        0
      );

    // Unique customers
    const customerIds =
      new Set(
        orders.map((order) =>
          order.customerId.toString()
        )
      );

    const totalCustomers =
      customerIds.size;

    // Order status counts
    const pendingOrders =
      orders.filter(
        (order) =>
          order.orderStatus === "placed"
      ).length;

    const processingOrders =
      orders.filter(
        (order) =>
          order.orderStatus ===
          "processing"
      ).length;

    const shippedOrders =
      orders.filter(
        (order) =>
          order.orderStatus === "shipped"
      ).length;

    const deliveredOrders =
      orders.filter(
        (order) =>
          order.orderStatus ===
          "delivered"
      ).length;

    const cancelledOrders =
      orders.filter(
        (order) =>
          order.orderStatus ===
          "cancelled"
      ).length;

    res.status(200).json({
      totalProducts,
      totalOrders,
      totalSales,
      totalCustomers,
      pendingOrders,
      processingOrders,
      shippedOrders,
      deliveredOrders,
      cancelledOrders
    });
  } catch (error) {
    res.status(500).json({
      message: error.message
    });
  }
};

module.exports = {
  getVendorAnalytics
};
Step 2 — Create Analytics Route

Create:

backend/routes/analyticsRoutes.js

Add:

const express = require("express");

const {
  getVendorAnalytics
} = require("../controllers/analyticsController");

const {
  protect,
  authorizeRoles
} = require("../middleware/authMiddleware");

const router = express.Router();

router.get(
  "/vendor",
  protect,
  authorizeRoles("vendor"),
  getVendorAnalytics
);

module.exports = router;

Your new API will be:

GET /api/analytics/vendor
Step 3 — Add Analytics Route to server.js

At the top:

const analyticsRoutes =
  require("./routes/analyticsRoutes");

Then add:

app.use(
  "/api/analytics",
  analyticsRoutes
);

So this section becomes:

app.use("/api/auth", authRoutes);
app.use("/api/users", userRoutes);
app.use("/api/stores", storeRoutes);
app.use("/api/products", productRoutes);
app.use("/api/upload", uploadRoutes);
app.use("/api/cart", cartRoutes);
app.use("/api/orders", orderRoutes);
app.use("/api/payments", paymentRoutes);
app.use("/api/analytics", analyticsRoutes);
Step 4 — Test Backend Analytics

Start your backend:

cd backend
npm run dev

Login as a vendor and get the JWT token.

You can test:

GET http://localhost:5000/api/analytics/vendor

with:

Authorization: Bearer YOUR_TOKEN

Expected response:

{
  "totalProducts": 5,
  "totalOrders": 12,
  "totalSales": 24500,
  "totalCustomers": 8,
  "pendingOrders": 2,
  "processingOrders": 3,
  "shippedOrders": 4,
  "deliveredOrders": 3,
  "cancelledOrders": 0
}

Your actual numbers will depend on your database.

Step 5 — Create Analytics Page

Create:

frontend/src/pages/Analytics.jsx

Add:

import { useEffect, useState } from "react";
import API from "../services/api";

function Analytics() {
  const [analytics, setAnalytics] =
    useState(null);

  const [loading, setLoading] =
    useState(true);

  const [message, setMessage] =
    useState("");

  const fetchAnalytics = async () => {
    try {
      const response =
        await API.get(
          "/analytics/vendor"
        );

      setAnalytics(response.data);
    } catch (error) {
      setMessage(
        error.response?.data?.message ||
        "Unable to load analytics"
      );
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchAnalytics();
  }, []);

  if (loading) {
    return (
      <div className="analytics-page">
        <h1>Analytics</h1>
        <p>Loading analytics...</p>
      </div>
    );
  }

  if (message) {
    return (
      <div className="analytics-page">
        <h1>Analytics</h1>
        <p>{message}</p>
      </div>
    );
  }

  return (
    <div className="analytics-page">
      <h1>Store Analytics</h1>

      <div className="analytics-grid">

        <div className="analytics-card">
          <h3>Total Products</h3>
          <p>
            {analytics.totalProducts}
          </p>
        </div>

        <div className="analytics-card">
          <h3>Total Orders</h3>
          <p>
            {analytics.totalOrders}
          </p>
        </div>

        <div className="analytics-card">
          <h3>Total Sales</h3>
          <p>
            ₹{analytics.totalSales}
          </p>
        </div>

        <div className="analytics-card">
          <h3>Total Customers</h3>
          <p>
            {analytics.totalCustomers}
          </p>
        </div>

      </div>

      <h2>Order Status</h2>

      <div className="analytics-grid">

        <div className="analytics-card">
          <h3>Placed</h3>
          <p>
            {analytics.pendingOrders}
          </p>
        </div>

        <div className="analytics-card">
          <h3>Processing</h3>
          <p>
            {analytics.processingOrders}
          </p>
        </div>

        <div className="analytics-card">
          <h3>Shipped</h3>
          <p>
            {analytics.shippedOrders}
          </p>
        </div>

        <div className="analytics-card">
          <h3>Delivered</h3>
          <p>
            {analytics.deliveredOrders}
          </p>
        </div>

        <div className="analytics-card">
          <h3>Cancelled</h3>
          <p>
            {analytics.cancelledOrders}
          </p>
        </div>

      </div>
    </div>
  );
}

export default Analytics;
Step 6 — Add Analytics Route

In App.jsx:

import Analytics from "./pages/Analytics";

Then:

<Route
  path="/analytics"
  element={
    <ProtectedRoute
      allowedRoles={["vendor"]}
    >
      <Analytics />
    </ProtectedRoute>
  }
/>
Step 7 — Add Analytics to Navbar

In the vendor section of Navbar.jsx:

<Link to="/analytics">
  Analytics
</Link>

Your vendor menu becomes:

Dashboard
Products
Inventory
Store
Orders
Analytics
Logout
Step 8 — Add CSS

Open:

frontend/src/index.css

Add:

.analytics-page {
  padding: 30px;
}

.analytics-grid {
  display: grid;
  grid-template-columns:
    repeat(auto-fit, minmax(200px, 1fr));

  gap: 20px;
  margin: 20px 0;
}

.analytics-card {
  padding: 20px;
  border-radius: 10px;
  background: white;
  border: 1px solid #ddd;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.analytics-card h3 {
  margin-bottom: 10px;
}

.analytics-card p {
  font-size: 28px;
  font-weight: bold;
}
Step 9 — What your dashboard will look like
             STORE ANALYTICS

┌────────────────┐ ┌────────────────┐
│ Total Products │ │ Total Orders   │
│      25        │ │      48        │
└────────────────┘ └────────────────┘

┌────────────────┐ ┌────────────────┐
│  Total Sales   │ │Total Customers │
│    ₹45,500     │ │      32        │
└────────────────┘ └────────────────┘


             ORDER STATUS

┌──────────┐ ┌────────────┐ ┌─────────┐
│  Placed  │ │ Processing │ │ Shipped │
│    5     │ │     10     │ │   12    │
└──────────┘ └────────────┘ └─────────┘

┌────────────┐ ┌───────────┐
│ Delivered  │ │ Cancelled │
│     21     │ │     0     │
└────────────┘ └───────────┘
Important security point

Notice that analytics uses:

const vendor = await User.findById(
  req.user.userId
).select("storeId");

and then:

Order.find({
  storeId
});

So analytics are tenant-specific.

For example:

Vendor A
   ↓
Store A
   ↓
Store A products/orders/sales

Vendor B
   ↓
Store B
   ↓
Store B products/orders/sales
