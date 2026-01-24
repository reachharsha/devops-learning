---
render_with_liquid: false
---
# 23 - Microservices Architecture with Docker

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand microservices architecture
- Design containerized microservices
- Implement service-to-service communication
- Handle data management in microservices
- Implement API gateway patterns
- Use service discovery
- Monitor distributed systems
- Deploy microservices with Docker

---

## 🏗️ Microservices vs Monolith

### **Monolithic Architecture:**
```
┌─────────────────────────────────┐
│      Single Application         │
│                                 │
│  ┌──────────────────────────┐  │
│  │   User Interface         │  │
│  ├──────────────────────────┤  │
│  │   Business Logic         │  │
│  ├──────────────────────────┤  │
│  │   Data Access Layer      │  │
│  └──────────────────────────┘  │
│              ↓                  │
│      ┌──────────────┐          │
│      │   Database   │          │
│      └──────────────┘          │
└─────────────────────────────────┘

Pros:
✅ Simple to develop
✅ Easy to test
✅ Easy to deploy

Cons:
❌ Tight coupling
❌ Hard to scale
❌ Long deployment cycles
❌ Technology lock-in
```

### **Microservices Architecture:**
```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Service A  │  │   Service B  │  │   Service C  │
│              │  │              │  │              │
│   ┌──────┐   │  │   ┌──────┐   │  │   ┌──────┐   │
│   │ API  │   │  │   │ API  │   │  │   │ API  │   │
│   ├──────┤   │  │   ├──────┤   │  │   ├──────┤   │
│   │Logic │   │  │   │Logic │   │  │   │Logic │   │
│   └──────┘   │  │   └──────┘   │  │   └──────┘   │
│      ↓       │  │      ↓       │  │      ↓       │
│   ┌──────┐   │  │   ┌──────┐   │  │   ┌──────┐   │
│   │  DB  │   │  │   │  DB  │   │  │   │  DB  │   │
│   └──────┘   │  │   └──────┘   │  │   └──────┘   │
└──────────────┘  └──────────────┘  └──────────────┘

Pros:
✅ Independent deployment
✅ Technology diversity
✅ Scalability
✅ Fault isolation

Cons:
❌ Distributed complexity
❌ Network overhead
❌ Data consistency
❌ Testing complexity
```

---

## 🚀 Example: E-Commerce Microservices

### **Architecture:**
```
                    ┌─────────────────┐
                    │   API Gateway   │
                    └────────┬────────┘
                             │
         ┌──────────────┬────┴────┬──────────────┐
         │              │         │              │
    ┌────▼────┐   ┌────▼────┐   ┌▼────┐   ┌─────▼─────┐
    │  User   │   │ Product │   │Order│   │  Payment  │
    │ Service │   │ Service │   │Svc  │   │  Service  │
    └─────────┘   └─────────┘   └─────┘   └───────────┘
         │             │            │            │
    ┌────▼────┐   ┌───▼─────┐  ┌──▼───┐   ┌────▼────┐
    │ User DB │   │Prod DB  │  │Order │   │Payment  │
    │(Postgres│   │(MongoDB)│  │ DB   │   │  DB     │
    └─────────┘   └─────────┘  └──────┘   └─────────┘
```

### **Project Structure:**
```bash
microservices-demo/
├── docker-compose.yml
├── api-gateway/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── user-service/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── product-service/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── order-service/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
└── payment-service/
    ├── Dockerfile
    ├── package.json
    └── server.js
```

---

## 🌐 API Gateway

### **Gateway Implementation:**
```javascript
// api-gateway/server.js
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');
const rateLimit = require('express-rate-limit');

const app = express();

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

app.use(limiter);
app.use(express.json());

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

// Route to User Service
app.use('/api/users', createProxyMiddleware({
  target: 'http://user-service:3001',
  changeOrigin: true,
  pathRewrite: { '^/api/users': '' }
}));

// Route to Product Service
app.use('/api/products', createProxyMiddleware({
  target: 'http://product-service:3002',
  changeOrigin: true,
  pathRewrite: { '^/api/products': '' }
}));

// Route to Order Service
app.use('/api/orders', createProxyMiddleware({
  target: 'http://order-service:3003',
  changeOrigin: true,
  pathRewrite: { '^/api/orders': '' }
}));

// Route to Payment Service
app.use('/api/payments', createProxyMiddleware({
  target: 'http://payment-service:3004',
  changeOrigin: true,
  pathRewrite: { '^/api/payments': '' }
}));

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`API Gateway running on port ${PORT}`);
});
```

### **Gateway Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"
CMD ["node", "server.js"]
```

---

## 👤 User Service

### **User Service Implementation:**
```javascript
// user-service/server.js
const express = require('express');
const { Pool } = require('pg');

const app = express();
app.use(express.json());

const pool = new Pool({
  host: process.env.DB_HOST || 'user-db',
  port: 5432,
  database: 'users',
  user: 'postgres',
  password: process.env.DB_PASSWORD || 'postgres'
});

// Initialize database
pool.query(`
  CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
  )
`);

// Get all users
app.get('/users', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM users');
    res.json(result.rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Get user by ID
app.get('/users/:id', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM users WHERE id = $1', [req.params.id]);
    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Create user
app.post('/users', async (req, res) => {
  try {
    const { email, name } = req.body;
    const result = await pool.query(
      'INSERT INTO users (email, name) VALUES ($1, $2) RETURNING *',
      [email, name]
    );
    res.status(201).json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Health check
app.get('/health', (req, res) => {
  pool.query('SELECT 1', (err) => {
    if (err) {
      return res.status(503).json({ status: 'unhealthy', error: err.message });
    }
    res.json({ status: 'healthy' });
  });
});

const PORT = 3001;
app.listen(PORT, () => {
  console.log(`User service running on port ${PORT}`);
});
```

---

## 📦 Product Service

### **Product Service with MongoDB:**
```javascript
// product-service/server.js
const express = require('express');
const mongoose = require('mongoose');

const app = express();
app.use(express.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGO_URL || 'mongodb://product-db:27017/products', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Product schema
const productSchema = new mongoose.Schema({
  name: { type: String, required: true },
  description: String,
  price: { type: Number, required: true },
  stock: { type: Number, default: 0 },
  createdAt: { type: Date, default: Date.now }
});

const Product = mongoose.model('Product', productSchema);

// Get all products
app.get('/products', async (req, res) => {
  try {
    const products = await Product.find();
    res.json(products);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Get product by ID
app.get('/products/:id', async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (!product) {
      return res.status(404).json({ error: 'Product not found' });
    }
    res.json(product);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Create product
app.post('/products', async (req, res) => {
  try {
    const product = new Product(req.body);
    await product.save();
    res.status(201).json(product);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Update stock
app.patch('/products/:id/stock', async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (!product) {
      return res.status(404).json({ error: 'Product not found' });
    }
    product.stock += req.body.quantity;
    await product.save();
    res.json(product);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Health check
app.get('/health', (req, res) => {
  const state = mongoose.connection.readyState;
  if (state === 1) {
    res.json({ status: 'healthy' });
  } else {
    res.status(503).json({ status: 'unhealthy' });
  }
});

const PORT = 3002;
app.listen(PORT, () => {
  console.log(`Product service running on port ${PORT}`);
});
```

---

## 🛒 Order Service

### **Order Service with Events:**
```javascript
// order-service/server.js
const express = require('express');
const axios = require('axios');
const { Pool } = require('pg');

const app = express();
app.use(express.json());

const pool = new Pool({
  host: process.env.DB_HOST || 'order-db',
  port: 5432,
  database: 'orders',
  user: 'postgres',
  password: process.env.DB_PASSWORD || 'postgres'
});

// Initialize database
pool.query(`
  CREATE TABLE IF NOT EXISTS orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL,
    total_price DECIMAL(10,2) NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
  )
`);

// Create order (orchestration pattern)
app.post('/orders', async (req, res) => {
  const { userId, productId, quantity } = req.body;
  
  try {
    // 1. Verify user exists
    const userResponse = await axios.get(`http://user-service:3001/users/${userId}`);
    if (userResponse.status !== 200) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    // 2. Get product details
    const productResponse = await axios.get(`http://product-service:3002/products/${productId}`);
    const product = productResponse.data;
    
    // 3. Check stock
    if (product.stock < quantity) {
      return res.status(400).json({ error: 'Insufficient stock' });
    }
    
    // 4. Calculate total
    const totalPrice = product.price * quantity;
    
    // 5. Create order
    const result = await pool.query(
      'INSERT INTO orders (user_id, product_id, quantity, total_price) VALUES ($1, $2, $3, $4) RETURNING *',
      [userId, productId, quantity, totalPrice]
    );
    const order = result.rows[0];
    
    // 6. Update product stock
    await axios.patch(`http://product-service:3002/products/${productId}/stock`, {
      quantity: -quantity
    });
    
    // 7. Process payment
    const paymentResponse = await axios.post('http://payment-service:3004/payments', {
      orderId: order.id,
      userId: userId,
      amount: totalPrice
    });
    
    if (paymentResponse.status === 201) {
      await pool.query('UPDATE orders SET status = $1 WHERE id = $2', ['completed', order.id]);
      order.status = 'completed';
    }
    
    res.status(201).json(order);
    
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Get all orders
app.get('/orders', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM orders');
    res.json(result.rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Health check
app.get('/health', (req, res) => {
  pool.query('SELECT 1', (err) => {
    if (err) {
      return res.status(503).json({ status: 'unhealthy' });
    }
    res.json({ status: 'healthy' });
  });
});

const PORT = 3003;
app.listen(PORT, () => {
  console.log(`Order service running on port ${PORT}`);
});
```

---

## 💳 Payment Service

### **Payment Service Implementation:**
```javascript
// payment-service/server.js
const express = require('express');
const { Pool } = require('pg');

const app = express();
app.use(express.json());

const pool = new Pool({
  host: process.env.DB_HOST || 'payment-db',
  port: 5432,
  database: 'payments',
  user: 'postgres',
  password: process.env.DB_PASSWORD || 'postgres'
});

pool.query(`
  CREATE TABLE IF NOT EXISTS payments (
    id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL,
    user_id INTEGER NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
  )
`);

// Process payment
app.post('/payments', async (req, res) => {
  try {
    const { orderId, userId, amount } = req.body;
    
    // Simulate payment processing
    const success = Math.random() > 0.1; // 90% success rate
    const status = success ? 'completed' : 'failed';
    
    const result = await pool.query(
      'INSERT INTO payments (order_id, user_id, amount, status) VALUES ($1, $2, $3, $4) RETURNING *',
      [orderId, userId, amount, status]
    );
    
    if (success) {
      res.status(201).json(result.rows[0]);
    } else {
      res.status(400).json({ error: 'Payment failed', payment: result.rows[0] });
    }
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Health check
app.get('/health', (req, res) => {
  pool.query('SELECT 1', (err) => {
    if (err) {
      return res.status(503).json({ status: 'unhealthy' });
    }
    res.json({ status: 'healthy' });
  });
});

const PORT = 3004;
app.listen(PORT, () => {
  console.log(`Payment service running on port ${PORT}`);
});
```

---

## 🐳 Docker Compose Configuration

### **Complete Stack:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  # API Gateway
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - user-service
      - product-service
      - order-service
      - payment-service
    networks:
      - frontend
      - backend
    restart: unless-stopped

  # User Service
  user-service:
    build: ./user-service
    environment:
      - DB_HOST=user-db
      - DB_PASSWORD=postgres
    depends_on:
      - user-db
    networks:
      - backend
    restart: unless-stopped

  user-db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=users
      - POSTGRES_PASSWORD=postgres
    volumes:
      - user-data:/var/lib/postgresql/data
    networks:
      - backend
    restart: unless-stopped

  # Product Service
  product-service:
    build: ./product-service
    environment:
      - MONGO_URL=mongodb://product-db:27017/products
    depends_on:
      - product-db
    networks:
      - backend
    restart: unless-stopped

  product-db:
    image: mongo:7
    volumes:
      - product-data:/data/db
    networks:
      - backend
    restart: unless-stopped

  # Order Service
  order-service:
    build: ./order-service
    environment:
      - DB_HOST=order-db
      - DB_PASSWORD=postgres
    depends_on:
      - order-db
    networks:
      - backend
    restart: unless-stopped

  order-db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=orders
      - POSTGRES_PASSWORD=postgres
    volumes:
      - order-data:/var/lib/postgresql/data
    networks:
      - backend
    restart: unless-stopped

  # Payment Service
  payment-service:
    build: ./payment-service
    environment:
      - DB_HOST=payment-db
      - DB_PASSWORD=postgres
    depends_on:
      - payment-db
    networks:
      - backend
    restart: unless-stopped

  payment-db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=payments
      - POSTGRES_PASSWORD=postgres
    volumes:
      - payment-data:/var/lib/postgresql/data
    networks:
      - backend
    restart: unless-stopped

networks:
  frontend:
  backend:

volumes:
  user-data:
  product-data:
  order-data:
  payment-data:
```

---

## 📊 Service Communication Patterns

### **1. Synchronous (REST):**
```javascript
// Direct HTTP call
const response = await axios.get('http://user-service:3001/users/123');
```

### **2. Asynchronous (Message Queue):**
```javascript
// With RabbitMQ
const amqp = require('amqplib');

// Publisher
const connection = await amqp.connect('amqp://rabbitmq');
const channel = await connection.createChannel();
await channel.assertQueue('orders');
channel.sendToQueue('orders', Buffer.from(JSON.stringify(order)));

// Consumer
channel.consume('orders', (msg) => {
  const order = JSON.parse(msg.content.toString());
  // Process order
  channel.ack(msg);
});
```

### **3. Event-Driven:**
```javascript
// Event emitter pattern
const EventEmitter = require('events');
const eventBus = new EventEmitter();

// Publisher
eventBus.emit('order.created', { orderId: 123, userId: 456 });

// Subscriber
eventBus.on('order.created', async (data) => {
  await sendConfirmationEmail(data.userId);
});
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the main benefit of microservices?**
   <details>
   <summary>Answer</summary>
   Independent deployment, scalability, fault isolation, technology diversity
   </details>

2. **What does an API Gateway do?**
   <details>
   <summary>Answer</summary>
   Single entry point, routing, rate limiting, authentication, load balancing
   </details>

3. **How do microservices communicate?**
   <details>
   <summary>Answer</summary>
   REST APIs, message queues, events, gRPC
   </details>

4. **What's a key challenge with microservices?**
   <details>
   <summary>Answer</summary>
   Distributed complexity, data consistency, network latency, monitoring
   </details>

5. **Why use different databases per service?**
   <details>
   <summary>Answer</summary>
   Loose coupling, independent scaling, technology choice per service
   </details>

---

## 📝 Key Takeaways

✅ **Microservices enable independent deployment**  
✅ **Each service has its own database**  
✅ **API Gateway provides single entry point**  
✅ **Services communicate via APIs or events**  
✅ **Docker simplifies microservices deployment**  
✅ **Service discovery enables dynamic routing**  
✅ **Health checks are critical**  
✅ **Distributed tracing helps debugging**  
✅ **Design for failure**  
✅ **Start simple, evolve to microservices**  

---

## 🚀 Next Steps

**Complete your Docker mastery:**

**Next lesson:** [24 - Production Checklist & Best Practices](24-production-checklist.md) - Final review and next steps

Microservices: Small services, big impact! 🚀
