**Last Updated:** November 2025  
**Author:** SUMITESHWAR KUMAR

---

# 🐱 **Mongoose ORM Complete Guide: From Beginner to Advanced**

## 📋 **Table of Contents**
- [What is Mongoose?](#what-is-mongoose)
- [Why Use Mongoose?](#why-use-mongoose)
- [Installation and Setup](#installation-and-setup)
- [Core Concepts](#core-concepts)
- [Schema Definition](#schema-definition)
- [Model Creation](#model-creation)
- [CRUD Operations](#crud-operations)
- [Query Methods](#query-methods)
- [Middleware](#middleware)
- [Validation](#validation)
- [Population](#population)
- [Plugins](#plugins)
- [Advanced Features](#advanced-features)
- [Performance Optimization](#performance-optimization)
- [Best Practices](#best-practices)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)
- [Migration Guide](#migration-guide)

---

## 🤔 **What is Mongoose?**

**Mongoose** is an **Object Data Modeling (ODM)** library for **MongoDB** and **Node.js**. It provides a higher-level abstraction over MongoDB's native driver, making it easier to work with MongoDB from Node.js applications.

### **Key Characteristics:**
- ✅ **Schema-based**: Define data structure and validation
- ✅ **Built-in type casting**: Automatic type conversion
- ✅ **Validation**: Built-in and custom validation rules
- ✅ **Query building**: Chainable query API
- ✅ **Middleware**: Pre/post hooks for operations
- ✅ **Plugins**: Extensible architecture
- ✅ **Population**: Automatic reference resolution

### **Architecture Overview:**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Application   │    │     Mongoose    │    │    MongoDB      │
│                 │────▶                 │────▶                 │
│ • Models        │    │ • Schemas       │    │ • Collections   │
│ • Queries       │    │ • Models        │    │ • Documents     │
│ • Validation    │    │ • Middleware    │    │ • Indexes       │
│ • Middleware    │    │ • Validation    │    │ • Aggregation   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## 🎯 **Why Use Mongoose?**

### **Problems Mongoose Solves:**

#### **1. Schema Definition and Validation**
```javascript
// ❌ WITHOUT MONGOOSE - Raw MongoDB
db.users.insertOne({
  name: "John",
  email: "john@", // Invalid email - no validation
  age: "25",      // String instead of number
  createdAt: "now" // Inconsistent date format
});

// ✅ WITH MONGOOSE - Structured and Validated
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, match: /.+\@.+\..+/ },
  age: { type: Number, min: 0, max: 120 },
  createdAt: { type: Date, default: Date.now }
});
```

#### **2. Type Safety and Casting**
```javascript
// ❌ WITHOUT MONGOOSE - Manual casting
const user = await db.users.findOne({ _id: req.params.id });
if (user) {
  user.age = parseInt(user.age); // Manual parsing
  user.createdAt = new Date(user.createdAt); // Manual casting
}

// ✅ WITH MONGOOSE - Automatic casting
const user = await User.findById(req.params.id);
// age is automatically a number
// createdAt is automatically a Date object
```

#### **3. Relationship Management**
```javascript
// ❌ WITHOUT MONGOOSE - Manual population
const post = await db.posts.findOne({ _id: postId });
const author = await db.users.findOne({ _id: post.authorId });
const comments = await db.comments.find({ postId: postId });

// ✅ WITH MONGOOSE - Automatic population
const post = await Post.findById(postId).populate('author').populate('comments');
// Single query with automatic joins
```

#### **4. Business Logic Encapsulation**
```javascript
// ❌ WITHOUT MONGOOSE - Business logic scattered
const user = await db.users.findOne({ email: req.body.email });
if (user) {
  // Password comparison logic here
  // Token generation logic here
  // Update logic here
}

// ✅ WITH MONGOOSE - Encapsulated in model
const user = await User.findOne({ email: req.body.email });
if (user) {
  const isValidPassword = await user.comparePassword(req.body.password);
  const token = user.generateAuthToken();
  await user.updateLastLogin();
}
```

---

## 📦 **Installation and Setup**

### **Basic Installation:**
```bash
# Install Mongoose
npm install mongoose

# Install MongoDB (if not already installed)
# macOS
brew install mongodb-community

# Ubuntu/Debian
sudo apt-get install mongodb

# Or use MongoDB Atlas (cloud)
```

### **Basic Connection:**
```javascript
const mongoose = require('mongoose');

// Connect to MongoDB
async function connectDB() {
  try {
    await mongoose.connect('mongodb://localhost:27017/myapp', {
      useNewUrlParser: true,
      useUnifiedTopology: true,
      useCreateIndex: true,
      useFindAndModify: false
    });

    console.log('MongoDB connected successfully');
  } catch (error) {
    console.error('MongoDB connection error:', error);
    process.exit(1);
  }
}

connectDB();
```

### **Environment-Based Configuration:**
```javascript
// config/database.js
const mongoose = require('mongoose');

const connectDB = async () => {
  const mongoURI = process.env.MONGODB_URI || 'mongodb://localhost:27017/myapp';

  const options = {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useCreateIndex: true,
    useFindAndModify: false,
    // Production options
    ...(process.env.NODE_ENV === 'production' && {
      ssl: true,
      sslValidate: true,
      sslCA: process.env.CA_CERT,
      auth: {
        username: process.env.DB_USER,
        password: process.env.DB_PASS
      },
      replicaSet: process.env.REPLICA_SET
    })
  };

  try {
    const conn = await mongoose.connect(mongoURI, options);
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error('Database connection error:', error.message);
    process.exit(1);
  }
};

module.exports = connectDB;
```

---

## 🏗️ **Core Concepts**

### **1. Schema**
A **Schema** defines the structure of documents in a MongoDB collection.

```javascript
const mongoose = require('mongoose');

// Define a schema
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true,
    maxlength: 50
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    match: [/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/, 'Invalid email']
  },
  age: {
    type: Number,
    min: 0,
    max: 120,
    default: 18
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  },
  isActive: {
    type: Boolean,
    default: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
  updatedAt: {
    type: Date,
    default: Date.now
  }
});

// Schema options
userSchema.set('toJSON', { virtuals: true });
userSchema.set('toObject', { virtuals: true });
```

### **2. Model**
A **Model** is a compiled version of a schema that provides an interface to interact with the database.

```javascript
// Create a model
const User = mongoose.model('User', userSchema);

// The model provides methods for database operations
const user = new User({ name: 'John', email: 'john@example.com' });
await user.save();

// Static methods
const users = await User.find({ age: { $gte: 18 } });

// Instance methods
const user = await User.findById('userId');
await user.updateLastLogin();
```

### **3. Document**
A **Document** is an instance of a model that represents a single database record.

```javascript
// Create a new document
const user = new User({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30
});

// Save to database
await user.save();

// Access document properties
console.log(user.name);        // 'John Doe'
console.log(user._id);         // MongoDB ObjectId
console.log(user.createdAt);   // Date object

// Update document
user.age = 31;
await user.save();

// Delete document
await user.remove();
```

### **4. Connection**
A **Connection** represents a connection to a MongoDB database.

```javascript
// Single connection
const conn = await mongoose.createConnection('mongodb://localhost:27017/myapp');

// Multiple connections
const db1 = mongoose.createConnection('mongodb://localhost:27017/db1');
const db2 = mongoose.createConnection('mongodb://localhost:27017/db2');

// Connection events
mongoose.connection.on('connected', () => console.log('Connected to MongoDB'));
mongoose.connection.on('error', (err) => console.error('Connection error:', err));
mongoose.connection.on('disconnected', () => console.log('Disconnected from MongoDB'));
```

---

## 📝 **Schema Definition**

### **Field Types:**
```javascript
const schema = new mongoose.Schema({
  // String
  name: String,
  description: { type: String, maxlength: 500 },

  // Number
  age: Number,
  price: { type: Number, min: 0, max: 10000 },

  // Boolean
  isActive: Boolean,
  isVerified: { type: Boolean, default: false },

  // Date
  createdAt: { type: Date, default: Date.now },
  birthday: Date,

  // ObjectId (for references)
  author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },

  // Arrays
  tags: [String],
  scores: [Number],
  comments: [{
    text: String,
    author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    createdAt: { type: Date, default: Date.now }
  }],

  // Mixed (flexible object)
  metadata: mongoose.Schema.Types.Mixed,

  // Buffer (for binary data)
  avatar: Buffer,

  // Custom types
  location: {
    type: { type: String, enum: ['Point'], default: 'Point' },
    coordinates: { type: [Number], default: [0, 0] }
  }
});
```

### **Schema Options:**
```javascript
const schema = new mongoose.Schema({
  // Field definitions
}, {
  // Schema options
  collection: 'custom_collection_name',    // Custom collection name
  timestamps: true,                        // Auto add createdAt/updatedAt
  versionKey: '__v',                       // Version key field name
  strict: true,                           // Strict mode (default: true)
  strictQuery: true,                      // Strict query mode
  minimize: false,                        // Don't minimize objects
  toJSON: { virtuals: true },             // Include virtuals in JSON
  toObject: { virtuals: true }            // Include virtuals in objects
});
```

### **Advanced Schema Features:**

#### **Virtual Properties:**
```javascript
// Virtual property for full name
userSchema.virtual('fullName').get(function() {
  return `${this.firstName} ${this.lastName}`;
});

// Virtual property for age calculation
userSchema.virtual('age').get(function() {
  return Math.floor((Date.now() - this.birthDate) / (365.25 * 24 * 60 * 60 * 1000));
});

// Setter virtual
userSchema.virtual('fullName').set(function(name) {
  const [firstName, lastName] = name.split(' ');
  this.firstName = firstName;
  this.lastName = lastName;
});
```

#### **Indexes:**
```javascript
// Single field index
userSchema.index({ email: 1 });

// Compound index
userSchema.index({ name: 1, age: -1 });

// Unique index
userSchema.index({ email: 1 }, { unique: true });

// Text index for search
userSchema.index({ name: 'text', description: 'text' });

// Geospatial index
userSchema.index({ location: '2dsphere' });

// TTL index (auto-expire documents)
userSchema.index({ createdAt: 1 }, { expireAfterSeconds: 3600 });
```

#### **Nested Schemas:**
```javascript
// Address sub-schema
const addressSchema = new mongoose.Schema({
  street: String,
  city: String,
  state: String,
  zipCode: String,
  country: { type: String, default: 'USA' }
});

// User schema with nested address
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  address: addressSchema,  // Nested schema
  billingAddress: addressSchema,  // Another nested schema
  shippingAddresses: [addressSchema]  // Array of nested schemas
});
```

---

## 🏭 **Model Creation**

### **Creating Models:**
```javascript
// Method 1: Direct model creation
const User = mongoose.model('User', userSchema);

// Method 2: Model with custom collection name
const Product = mongoose.model('Product', productSchema, 'products_collection');

// Method 3: Model from existing connection
const db = mongoose.createConnection('mongodb://localhost:27017/myapp');
const User = db.model('User', userSchema);
```

### **Model Methods:**

#### **Static Methods:**
```javascript
// Add static method to schema
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

userSchema.statics.findActiveUsers = function() {
  return this.find({ isActive: true });
};

userSchema.statics.getUserStats = function() {
  return this.aggregate([
    { $group: { _id: '$role', count: { $sum: 1 } } },
    { $sort: { count: -1 } }
  ]);
};

// Usage
const user = await User.findByEmail('john@example.com');
const activeUsers = await User.findActiveUsers();
const stats = await User.getUserStats();
```

#### **Instance Methods:**
```javascript
// Add instance methods to schema
userSchema.methods.getFullName = function() {
  return `${this.firstName} ${this.lastName}`;
};

userSchema.methods.generateAuthToken = function() {
  const token = jwt.sign({ _id: this._id }, process.env.JWT_SECRET);
  return token;
};

userSchema.methods.comparePassword = async function(candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

userSchema.methods.updateLastLogin = function() {
  this.lastLogin = new Date();
  return this.save();
};

// Usage
const user = await User.findById(userId);
console.log(user.getFullName());
const token = user.generateAuthToken();
await user.updateLastLogin();
```

#### **Query Helpers:**
```javascript
// Add query helpers
userSchema.query.byName = function(name) {
  return this.where({ name: new RegExp(name, 'i') });
};

userSchema.query.active = function() {
  return this.where({ isActive: true });
};

userSchema.query.byAgeRange = function(min, max) {
  return this.where('age').gte(min).lte(max);
};

// Usage
const users = await User.find().byName('john').active().byAgeRange(18, 65);
```

---

## 🔄 **CRUD Operations**

### **Create Operations:**

#### **1. Creating Documents:**
```javascript
// Method 1: Using constructor + save()
const user = new User({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30
});
await user.save();

// Method 2: Using create() - saves automatically
const user = await User.create({
  name: 'Jane Doe',
  email: 'jane@example.com',
  age: 25
});

// Method 3: Bulk create
const users = await User.insertMany([
  { name: 'User 1', email: 'user1@example.com' },
  { name: 'User 2', email: 'user2@example.com' },
  { name: 'User 3', email: 'user3@example.com' }
]);
```

#### **2. Validation on Create:**
```javascript
try {
  const user = await User.create({
    name: '',  // Will fail required validation
    email: 'invalid-email',  // Will fail regex validation
    age: 150  // Will fail max validation
  });
} catch (error) {
  if (error.name === 'ValidationError') {
    console.log('Validation errors:', error.errors);
  }
}
```

### **Read Operations:**

#### **1. Find Operations:**
```javascript
// Find all documents
const users = await User.find();

// Find with conditions
const activeUsers = await User.find({ isActive: true });

// Find one document
const user = await User.findOne({ email: 'john@example.com' });

// Find by ID
const user = await User.findById('507f1f77bcf86cd799439011');

// Find with projection (select specific fields)
const users = await User.find({}, 'name email age');

// Find with sorting
const users = await User.find().sort({ createdAt: -1 });

// Find with pagination
const users = await User.find()
  .sort({ createdAt: -1 })
  .limit(10)
  .skip(20);
```

#### **2. Advanced Queries:**
```javascript
// Query with regex
const users = await User.find({
  name: { $regex: 'john', $options: 'i' }
});

// Query with comparison operators
const users = await User.find({
  age: { $gte: 18, $lte: 65 },
  createdAt: { $gte: new Date('2023-01-01') }
});

// Query with logical operators
const users = await User.find({
  $or: [
    { age: { $lt: 18 } },
    { age: { $gt: 65 } }
  ],
  isActive: true
});

// Query with array operations
const users = await User.find({
  tags: { $in: ['javascript', 'nodejs'] },
  scores: { $elemMatch: { $gte: 80 } }
});
```

### **Update Operations:**

#### **1. Update Methods:**
```javascript
// Update one document
await User.updateOne(
  { email: 'john@example.com' },
  { $set: { lastLogin: new Date() }, $inc: { loginCount: 1 } }
);

// Update multiple documents
await User.updateMany(
  { isActive: false },
  { $set: { deactivatedAt: new Date() } }
);

// Find and update (returns updated document)
const user = await User.findOneAndUpdate(
  { email: 'john@example.com' },
  { $set: { lastLogin: new Date() } },
  { new: true, runValidators: true }
);

// Update by ID
const user = await User.findByIdAndUpdate(
  '507f1f77bcf86cd799439011',
  { $set: { name: 'John Smith' } },
  { new: true }
);
```

#### **2. Instance Updates:**
```javascript
// Update instance and save
const user = await User.findById('507f1f77bcf86cd799439011');
user.name = 'John Smith';
user.age = 31;
await user.save();

// Update with validation
try {
  await user.save();
} catch (error) {
  if (error.name === 'ValidationError') {
    console.log('Validation failed:', error.errors);
  }
}
```

### **Delete Operations:**

#### **1. Delete Methods:**
```javascript
// Delete one document
await User.deleteOne({ email: 'john@example.com' });

// Delete multiple documents
await User.deleteMany({ isActive: false });

// Find and delete (returns deleted document)
const deletedUser = await User.findOneAndDelete({
  email: 'john@example.com'
});

// Delete by ID
const deletedUser = await User.findByIdAndDelete(
  '507f1f77bcf86cd799439011'
);
```

#### **2. Soft Delete Pattern:**
```javascript
// Add soft delete to schema
userSchema.add({
  deletedAt: { type: Date, default: null },
  isDeleted: { type: Boolean, default: false }
});

// Soft delete method
userSchema.methods.softDelete = function() {
  this.isDeleted = true;
  this.deletedAt = new Date();
  return this.save();
};

// Soft delete static
userSchema.statics.softDelete = function(id) {
  return this.findByIdAndUpdate(id, {
    isDeleted: true,
    deletedAt: new Date()
  });
};

// Query helper for non-deleted documents
userSchema.pre('find', function() {
  this.where({ isDeleted: { $ne: true } });
});
```

---

## 🔍 **Query Methods**

### **Query Building:**
```javascript
// Chainable query methods
const users = await User.find()
  .where('age').gte(18).lte(65)  // Age between 18-65
  .where('isActive').equals(true)  // Only active users
  .select('name email age')        // Select specific fields
  .sort({ createdAt: -1 })         // Sort by creation date (desc)
  .limit(10)                       // Limit to 10 results
  .skip(20)                        // Skip first 20 results
  .populate('posts')               // Populate referenced documents
  .exec();                         // Execute the query
```

### **Advanced Query Methods:**

#### **1. Aggregation Framework:**
```javascript
// Basic aggregation
const stats = await User.aggregate([
  { $match: { isActive: true } },
  { $group: {
    _id: '$role',
    count: { $sum: 1 },
    averageAge: { $avg: '$age' }
  }},
  { $sort: { count: -1 } }
]);

// Complex aggregation with lookup
const usersWithPosts = await User.aggregate([
  { $match: { isActive: true } },
  { $lookup: {
    from: 'posts',
    localField: '_id',
    foreignField: 'author',
    as: 'posts'
  }},
  { $addFields: {
    postCount: { $size: '$posts' }
  }},
  { $sort: { postCount: -1 } },
  { $limit: 10 }
]);
```

#### **2. Text Search:**
```javascript
// Create text index
userSchema.index({ name: 'text', bio: 'text' });

// Text search
const users = await User.find({
  $text: { $search: 'john javascript developer' }
}, {
  score: { $meta: 'textScore' }
}).sort({
  score: { $meta: 'textScore' }
});
```

#### **3. Geospatial Queries:**
```javascript
// Create geospatial index
userSchema.index({ location: '2dsphere' });

// Find users within radius
const users = await User.find({
  location: {
    $near: {
      $geometry: {
        type: 'Point',
        coordinates: [-73.9667, 40.78]  // [longitude, latitude]
      },
      $maxDistance: 1000  // 1km radius
    }
  }
});

// Find users within polygon
const users = await User.find({
  location: {
    $geoWithin: {
      $geometry: {
        type: 'Polygon',
        coordinates: [[
          [-74.0, 40.7],
          [-73.9, 40.7],
          [-73.9, 40.8],
          [-74.0, 40.8],
          [-74.0, 40.7]
        ]]
      }
    }
  }
});
```

#### **4. Streaming Queries:**
```javascript
// Stream large result sets
const cursor = User.find().cursor();

cursor.on('data', (doc) => {
  console.log('Received document:', doc.name);
});

cursor.on('end', () => {
  console.log('Stream ended');
});

cursor.on('error', (err) => {
  console.error('Stream error:', err);
});
```

---

## 🎣 **Middleware**

### **Pre Middleware:**
```javascript
// Pre-save middleware
userSchema.pre('save', async function(next) {
  // Hash password before saving
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 12);
  }

  // Update timestamp
  this.updatedAt = new Date();

  next();
});

// Pre-find middleware
userSchema.pre('find', function(next) {
  // Add default conditions
  this.where({ isDeleted: { $ne: true } });
  next();
});

// Pre-remove middleware
userSchema.pre('remove', async function(next) {
  // Clean up related documents
  await Comment.deleteMany({ author: this._id });
  await Post.updateMany(
    { author: this._id },
    { $set: { author: null } }
  );

  next();
});
```

### **Post Middleware:**
```javascript
// Post-save middleware
userSchema.post('save', function(doc, next) {
  console.log(`User ${doc.name} has been saved`);

  // Send welcome email (don't block save operation)
  sendWelcomeEmail(doc.email).catch(err => {
    console.error('Failed to send welcome email:', err);
  });

  next();
});

// Post-find middleware
userSchema.post('find', function(docs, next) {
  console.log(`Found ${docs.length} documents`);
  next();
});

// Post-error middleware
userSchema.post('save', function(error, doc, next) {
  if (error.name === 'MongoError' && error.code === 11000) {
    next(new Error('Email address already exists'));
  } else {
    next(error);
  }
});
```

### **Advanced Middleware Patterns:**

#### **1. Audit Trail Middleware:**
```javascript
// Add audit fields to schema
userSchema.add({
  createdBy: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  updatedBy: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  auditLog: [{
    action: String,
    timestamp: { type: Date, default: Date.now },
    user: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    changes: mongoose.Schema.Types.Mixed
  }]
});

// Audit middleware
userSchema.pre('save', function(next) {
  if (this.isNew) {
    // Track creation
    this.auditLog.push({
      action: 'created',
      user: this.createdBy,
      changes: this.toObject()
    });
  } else {
    // Track updates
    const changes = {};
    this.modifiedPaths().forEach(path => {
      changes[path] = {
        from: this.originalValue(path),
        to: this.get(path)
      };
    });

    if (Object.keys(changes).length > 0) {
      this.auditLog.push({
        action: 'updated',
        user: this.updatedBy,
        changes
      });
    }
  }

  next();
});
```

#### **2. Caching Middleware:**
```javascript
const cache = require('memory-cache');

// Cache middleware
userSchema.post('find', function(docs) {
  if (Array.isArray(docs)) {
    docs.forEach(doc => {
      cache.put(`user:${doc._id}`, doc, 300000); // Cache for 5 minutes
    });
  }
});

userSchema.post('findOne', function(doc) {
  if (doc) {
    cache.put(`user:${doc._id}`, doc, 300000);
  }
});

// Custom static method with caching
userSchema.statics.findByIdCached = async function(id) {
  const cacheKey = `user:${id}`;
  let user = cache.get(cacheKey);

  if (!user) {
    user = await this.findById(id);
    if (user) {
      cache.put(cacheKey, user, 300000);
    }
  }

  return user;
};
```

---

## ✅ **Validation**

### **Built-in Validators:**
```javascript
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    minlength: [2, 'Name must be at least 2 characters'],
    maxlength: [50, 'Name cannot exceed 50 characters'],
    trim: true
  },

  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    match: [/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/, 'Invalid email format']
  },

  age: {
    type: Number,
    min: [0, 'Age cannot be negative'],
    max: [120, 'Age cannot exceed 120'],
    required: [true, 'Age is required']
  },

  role: {
    type: String,
    enum: {
      values: ['user', 'admin', 'moderator'],
      message: 'Role must be user, admin, or moderator'
    },
    default: 'user'
  }
});
```

### **Custom Validators:**
```javascript
// Custom validator function
function validateEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

// Custom validator with async
async function isEmailUnique(email) {
  const existingUser = await mongoose.models.User.findOne({ email });
  return !existingUser;
}

// Apply custom validators
userSchema.path('email').validate({
  validator: validateEmail,
  message: 'Invalid email format'
});

userSchema.path('email').validate({
  validator: isEmailUnique,
  message: 'Email address already exists',
  isAsync: true
});
```

### **Document-level Validation:**
```javascript
// Document-level validation
userSchema.pre('validate', function(next) {
  if (this.age < 13 && this.role === 'admin') {
    this.invalidate('role', 'Users under 13 cannot be admins');
  }

  if (this.firstName && this.lastName) {
    this.fullName = `${this.firstName} ${this.lastName}`;
  }

  next();
});

// Cross-field validation
userSchema.pre('save', function(next) {
  if (this.password !== this.confirmPassword) {
    const error = new Error('Passwords do not match');
    error.statusCode = 400;
    return next(error);
  }

  next();
});
```

### **Conditional Validation:**
```javascript
// Conditional required fields
userSchema.path('businessLicense').validate(function(value) {
  if (this.accountType === 'business' && !value) {
    return false;
  }
  return true;
}, 'Business license is required for business accounts');

// Dynamic validation based on other fields
userSchema.path('age').validate(function(value) {
  if (this.accountType === 'senior' && value < 65) {
    return false;
  }
  return true;
}, 'Senior accounts require age 65 or older');
```

### **Custom Error Messages:**
```javascript
// Custom error messages
const customMessages = {
  'name.required': 'Please provide a name',
  'email.required': 'Email address is mandatory',
  'email.unique': 'This email is already registered',
  'age.min': 'Age must be at least {MIN}',
  'age.max': 'Age cannot exceed {MAX}'
};

// Apply custom messages
for (const [path, message] of Object.entries(customMessages)) {
  const [field, validator] = path.split('.');
  userSchema.path(field).validators.forEach(validatorObj => {
    if (validatorObj.type === validator) {
      validatorObj.message = message;
    }
  });
}
```

---

## 🔗 **Population**

### **Basic Population:**
```javascript
// Define schemas with references
const authorSchema = new mongoose.Schema({
  name: String,
  email: String
});

const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  tags: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Tag' }],
  createdAt: { type: Date, default: Date.now }
});

// Populate single reference
const posts = await Post.find().populate('author');

// Populate multiple references
const posts = await Post.find().populate('author').populate('tags');

// Populate with specific fields
const posts = await Post.find().populate('author', 'name email');

// Populate nested references
const posts = await Post.find().populate({
  path: 'author',
  populate: {
    path: 'profile',
    select: 'avatar bio'
  }
});
```

### **Advanced Population:**

#### **1. Conditional Population:**
```javascript
// Populate based on conditions
const posts = await Post.find()
  .populate({
    path: 'author',
    match: { isActive: true },  // Only populate active authors
    select: 'name email'
  })
  .populate({
    path: 'comments',
    match: { isApproved: true }, // Only approved comments
    options: { sort: { createdAt: -1 } }
  });
```

#### **2. Virtual Population:**
```javascript
// Virtual populate (reverse references)
userSchema.virtual('posts', {
  ref: 'Post',          // Model to populate from
  localField: '_id',    // Field in User model
  foreignField: 'author' // Field in Post model
});

// Enable virtuals in JSON
userSchema.set('toJSON', { virtuals: true });
userSchema.set('toObject', { virtuals: true });

// Usage
const user = await User.findById(userId).populate('posts');
```

#### **3. Dynamic Population:**
```javascript
// Dynamic population based on query parameters
async function getPostsWithDynamicPopulation(query = {}) {
  const populateOptions = [];

  if (query.includeAuthor) {
    populateOptions.push({
      path: 'author',
      select: query.authorFields || 'name email'
    });
  }

  if (query.includeComments) {
    populateOptions.push({
      path: 'comments',
      populate: {
        path: 'author',
        select: 'name'
      }
    });
  }

  if (query.includeTags) {
    populateOptions.push('tags');
  }

  let postsQuery = Post.find();

  populateOptions.forEach(option => {
    postsQuery = postsQuery.populate(option);
  });

  return await postsQuery.exec();
}
```

#### **4. Population with Aggregation:**
```javascript
// Population using aggregation pipeline
const usersWithPosts = await User.aggregate([
  {
    $lookup: {
      from: 'posts',
      localField: '_id',
      foreignField: 'author',
      as: 'posts'
    }
  },
  {
    $addFields: {
      postCount: { $size: '$posts' },
      latestPost: { $arrayElemAt: ['$posts', 0] }
    }
  },
  {
    $sort: { postCount: -1 }
  }
]);
```

### **Population Middleware:**
```javascript
// Auto-populate middleware
postSchema.pre('find', function() {
  this.populate('author', 'name email');
});

postSchema.pre('findOne', function() {
  this.populate('author', 'name email');
});

// Conditional auto-populate
function autoPopulate(next) {
  this.populate('author', 'name email');
  this.populate('tags', 'name color');
  next();
}

postSchema.pre('find', autoPopulate);
postSchema.pre('findOne', autoPopulate);
```

---

## 🔌 **Plugins**

### **Popular Built-in Plugins:**

#### **1. Passport Local Mongoose:**
```javascript
const passportLocalMongoose = require('passport-local-mongoose');

userSchema.plugin(passportLocalMongoose, {
  usernameField: 'email',
  attemptsField: 'loginAttempts',
  lastLoginField: 'lastLogin',
  lockUntilField: 'lockUntil'
});

// Adds methods like authenticate, register, etc.
```

#### **2. Mongoose Paginate:**
```javascript
const mongoosePaginate = require('mongoose-paginate-v2');

userSchema.plugin(mongoosePaginate);

// Usage
const options = {
  page: 1,
  limit: 10,
  sort: { createdAt: -1 }
};

const result = await User.paginate({}, options);
// Returns: { docs, totalDocs, limit, page, totalPages, hasNextPage, hasPrevPage }
```

#### **3. Mongoose Autopopulate:**
```javascript
const autopopulate = require('mongoose-autopopulate');

postSchema.plugin(autopopulate);

const postSchema = new mongoose.Schema({
  title: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    autopopulate: true  // Auto-populate this field
  }
});
```

### **Custom Plugin Development:**

#### **1. Basic Plugin Structure:**
```javascript
// Custom plugin for soft delete
function softDeletePlugin(schema, options = {}) {
  // Add fields to schema
  schema.add({
    deletedAt: { type: Date, default: null },
    isDeleted: { type: Boolean, default: false }
  });

  // Add instance methods
  schema.methods.softDelete = function() {
    this.isDeleted = true;
    this.deletedAt = new Date();
    return this.save();
  };

  schema.methods.restore = function() {
    this.isDeleted = false;
    this.deletedAt = null;
    return this.save();
  };

  // Add static methods
  schema.statics.findDeleted = function() {
    return this.find({ isDeleted: true });
  };

  schema.statics.findNotDeleted = function() {
    return this.find({ isDeleted: { $ne: true } });
  };

  // Add pre middleware
  schema.pre('find', function() {
    if (options.includeDeleted !== true) {
      this.where({ isDeleted: { $ne: true } });
    }
  });

  schema.pre('findOne', function() {
    if (options.includeDeleted !== true) {
      this.where({ isDeleted: { $ne: true } });
    }
  });

  schema.pre('countDocuments', function() {
    if (options.includeDeleted !== true) {
      this.where({ isDeleted: { $ne: true } });
    }
  });
}

// Usage
userSchema.plugin(softDeletePlugin, { includeDeleted: false });
```

#### **2. Advanced Plugin with Options:**
```javascript
// Plugin for audit trail
function auditPlugin(schema, options = {}) {
  const auditFields = options.fields || ['createdBy', 'updatedBy', 'createdAt', 'updatedAt'];

  // Add audit fields
  const auditFieldDefinitions = {};
  auditFields.forEach(field => {
    if (field === 'createdAt' || field === 'updatedAt') {
      auditFieldDefinitions[field] = { type: Date, default: Date.now };
    } else {
      auditFieldDefinitions[field] = {
        type: mongoose.Schema.Types.ObjectId,
        ref: options.userModel || 'User'
      };
    }
  });

  schema.add(auditFieldDefinitions);

  // Pre-save middleware for audit
  schema.pre('save', function(next) {
    const userId = options.getCurrentUser ? options.getCurrentUser() : null;

    if (this.isNew && auditFields.includes('createdBy')) {
      this.createdBy = userId;
    }

    if (auditFields.includes('updatedBy')) {
      this.updatedBy = userId;
    }

    if (auditFields.includes('updatedAt')) {
      this.updatedAt = new Date();
    }

    next();
  });

  // Pre-update middleware
  schema.pre('findOneAndUpdate', function(next) {
    const userId = options.getCurrentUser ? options.getCurrentUser() : null;

    if (auditFields.includes('updatedBy')) {
      this.set({ updatedBy: userId, updatedAt: new Date() });
    } else if (auditFields.includes('updatedAt')) {
      this.set({ updatedAt: new Date() });
    }

    next();
  });
}

// Usage
userSchema.plugin(auditPlugin, {
  fields: ['createdBy', 'updatedBy', 'createdAt', 'updatedAt'],
  userModel: 'User',
  getCurrentUser: () => req.user ? req.user._id : null
});
```

---

## ⚡ **Advanced Features**

### **Transactions:**
```javascript
// Start a session
const session = await mongoose.startSession();

// Start transaction
session.startTransaction();

try {
  // Perform multiple operations
  const user = await User.create([{
    name: 'John Doe',
    email: 'john@example.com'
  }], { session });

  const post = await Post.create([{
    title: 'My First Post',
    content: 'Hello World!',
    author: user[0]._id
  }], { session });

  // Update user with post reference
  await User.findByIdAndUpdate(
    user[0]._id,
    { $push: { posts: post[0]._id } },
    { session }
  );

  // Commit transaction
  await session.commitTransaction();
  console.log('Transaction committed successfully');

} catch (error) {
  // Abort transaction on error
  await session.abortTransaction();
  console.error('Transaction aborted:', error);

} finally {
  // End session
  session.endSession();
}
```

### **Connection Pooling:**
```javascript
const mongoose = require('mongoose');

// Configure connection pool
const options = {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  maxPoolSize: 10,      // Maximum number of connections in pool
  minPoolSize: 5,       // Minimum number of connections in pool
  maxIdleTimeMS: 30000, // Close connections after 30 seconds of inactivity
  socketTimeoutMS: 45000, // Close sockets after 45 seconds of inactivity
  serverSelectionTimeoutMS: 5000, // Keep trying to send operations for 5 seconds
  bufferCommands: false, // Disable mongoose buffering
  bufferMaxEntries: 0    // Disable mongoose buffering
};

mongoose.connect('mongodb://localhost:27017/myapp', options);
```

### **Change Streams (Real-time):**
```javascript
// Watch for changes on a collection
const changeStream = User.watch();

// Listen for change events
changeStream.on('change', (change) => {
  console.log('Change detected:', change.operationType);

  switch (change.operationType) {
    case 'insert':
      console.log('New user created:', change.fullDocument.name);
      break;
    case 'update':
      console.log('User updated:', change.documentKey._id);
      break;
    case 'delete':
      console.log('User deleted:', change.documentKey._id);
      break;
  }
});

// Watch with pipeline (filter changes)
const filteredStream = User.watch([
  { $match: { 'fullDocument.isActive': true } },
  { $match: { operationType: 'update' } }
]);

filteredStream.on('change', (change) => {
  console.log('Active user updated:', change.fullDocument.name);
});
```

### **GridFS (Large Files):**
```javascript
const mongoose = require('mongoose');
const Grid = require('gridfs-stream');
const fs = require('fs');

// Create GridFS stream
const conn = mongoose.connection;
let gfs;

conn.once('open', () => {
  gfs = Grid(conn.db, mongoose.mongo);
  gfs.collection('uploads');
});

// Upload file to GridFS
const uploadFile = (filePath, filename) => {
  return new Promise((resolve, reject) => {
    const writestream = gfs.createWriteStream({
      filename: filename,
      content_type: 'image/jpeg'
    });

    fs.createReadStream(filePath)
      .pipe(writestream)
      .on('close', (file) => resolve(file))
      .on('error', reject);
  });
};

// Download file from GridFS
const downloadFile = (fileId, outputPath) => {
  return new Promise((resolve, reject) => {
    const readstream = gfs.createReadStream({ _id: fileId });
    const writestream = fs.createWriteStream(outputPath);

    readstream.pipe(writestream)
      .on('finish', resolve)
      .on('error', reject);
  });
};
```

### **Custom Types:**
```javascript
// Custom schema type for encrypted strings
class EncryptedString extends mongoose.SchemaType {
  constructor(key, options) {
    super(key, options, 'EncryptedString');
  }

  cast(val) {
    if (typeof val !== 'string') {
      throw new Error('EncryptedString: ' + val + ' is not a string');
    }
    return encrypt(val); // Your encryption function
  }

  doValidate(value, callback) {
    // Custom validation logic
    callback(value.length > 0);
  }
}

// Register custom type
mongoose.Schema.Types.EncryptedString = EncryptedString;

// Usage in schema
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  ssn: EncryptedString  // Will automatically encrypt/decrypt
});
```

---

## 🎯 **Best Practices**

### **Schema Design:**

#### **1. Use Proper Data Types:**
```javascript
// ✅ GOOD: Proper types
const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
  isActive: Boolean,
  createdAt: Date,
  tags: [String],
  profile: {
    avatar: String,
    bio: String
  }
});

// ❌ BAD: Everything as string
const userSchema = new mongoose.Schema({
  name: String,
  age: String,        // Should be Number
  isActive: String,   // Should be Boolean
  createdAt: String,  // Should be Date
  tags: String,       // Should be [String]
  profile: String     // Should be Object
});
```

#### **2. Define Indexes Strategically:**
```javascript
// ✅ GOOD: Strategic indexes
userSchema.index({ email: 1 }, { unique: true });        // Unique constraint
userSchema.index({ name: 1, age: -1 });                  // Compound index
userSchema.index({ location: '2dsphere' });              // Geospatial index
userSchema.index({ createdAt: 1 }, { expireAfterSeconds: 3600 }); // TTL index

// ❌ BAD: Too many indexes
userSchema.index({ name: 1 });
userSchema.index({ email: 1 });
userSchema.index({ age: 1 });
userSchema.index({ createdAt: 1 });
userSchema.index({ name: 1, email: 1 });  // Redundant with single indexes
```

#### **3. Use References Wisely:**
```javascript
// ✅ GOOD: Reference for large datasets
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  tags: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Tag' }]
});

// ❌ BAD: Embedding everything
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: {        // Embedded - bad if author data is large
    name: String,
    email: String,
    avatar: String,
    bio: String,
    followers: [String]  // Very large array
  }
});
```

### **Query Optimization:**

#### **1. Use Select to Limit Fields:**
```javascript
// ✅ GOOD: Select only needed fields
const users = await User.find({}, 'name email age');

// ❌ BAD: Select all fields
const users = await User.find(); // Returns all fields including large objects
```

#### **2. Use Lean Queries for Read-Only:**
```javascript
// ✅ GOOD: Use lean for read-only operations
const users = await User.find().lean(); // Returns plain objects, faster

// ❌ BAD: Use full Mongoose documents for read-only
const users = await User.find(); // Creates Mongoose documents with all overhead
```

#### **3. Batch Operations:**
```javascript
// ✅ GOOD: Batch operations
const users = [
  { name: 'User 1', email: 'user1@example.com' },
  { name: 'User 2', email: 'user2@example.com' }
];
await User.insertMany(users); // Single bulk operation

// ❌ BAD: Individual operations
for (const user of users) {
  await User.create(user); // Multiple round trips
}
```

### **Error Handling:**

#### **1. Proper Error Handling:**
```javascript
// ✅ GOOD: Comprehensive error handling
app.post('/users', async (req, res) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json(user);
  } catch (error) {
    if (error.name === 'ValidationError') {
      return res.status(400).json({
        error: 'Validation Error',
        details: Object.values(error.errors).map(err => err.message)
      });
    }

    if (error.code === 11000) {
      return res.status(409).json({
        error: 'Duplicate Error',
        message: 'Email address already exists'
      });
    }

    console.error('Unexpected error:', error);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// ❌ BAD: Poor error handling
app.post('/users', async (req, res) => {
  try {
    const user = await User.create(req.body);
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: 'Something went wrong' });
  }
});
```

#### **2. Connection Error Handling:**
```javascript
// ✅ GOOD: Connection error handling
mongoose.connection.on('error', (err) => {
  console.error('MongoDB connection error:', err);
  process.exit(1);
});

mongoose.connection.on('disconnected', () => {
  console.log('MongoDB disconnected. Attempting to reconnect...');
});

mongoose.connection.on('reconnected', () => {
  console.log('MongoDB reconnected');
});

// Graceful shutdown
process.on('SIGINT', async () => {
  console.log('Received SIGINT. Closing MongoDB connection...');
  await mongoose.connection.close();
  process.exit(0);
});
```

### **Security Best Practices:**

#### **1. Input Validation and Sanitization:**
```javascript
// ✅ GOOD: Input validation
const createUserSchema = Joi.object({
  name: Joi.string().min(2).max(50).required(),
  email: Joi.string().email().required(),
  age: Joi.number().min(0).max(120).required()
});

app.post('/users', async (req, res) => {
  const { error, value } = createUserSchema.validate(req.body);
  if (error) {
    return res.status(400).json({ error: error.details[0].message });
  }

  const user = await User.create(value);
  res.json(user);
});
```

#### **2. Prevent NoSQL Injection:**
```javascript
// ✅ GOOD: Safe queries
const user = await User.findOne({ email: req.body.email });

// ❌ BAD: Vulnerable to injection
const user = await User.findOne({ $where: `this.email === '${req.body.email}'` });
```

#### **3. Rate Limiting:**
```javascript
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later.'
});

app.use('/api/', apiLimiter);
```

### **Performance Optimization:**

#### **1. Connection Pooling:**
```javascript
// ✅ GOOD: Proper connection configuration
const options = {
  maxPoolSize: 10,
  minPoolSize: 5,
  maxIdleTimeMS: 30000,
  socketTimeoutMS: 45000,
  serverSelectionTimeoutMS: 5000
};

mongoose.connect(uri, options);
```

#### **2. Schema Optimization:**
```javascript
// ✅ GOOD: Optimized schema
const userSchema = new mongoose.Schema({
  name: { type: String, index: true },
  email: { type: String, unique: true, index: true },
  age: Number,
  createdAt: { type: Date, default: Date.now, index: true }
}, {
  // Enable timestamps
  timestamps: true,

  // Use toJSON and toObject to customize output
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});
```

#### **3. Query Optimization:**
```javascript
// ✅ GOOD: Optimized queries
const users = await User.find({ age: { $gte: 18 } })
  .select('name email age')  // Select only needed fields
  .sort({ createdAt: -1 })   // Use index for sorting
  .limit(10)                 // Limit results
  .lean();                   // Use lean queries

// Create compound index to support query
userSchema.index({ age: 1, createdAt: -1 });
```

---

## 🔧 **Common Patterns**

### **Repository Pattern:**
```javascript
// repositories/userRepository.js
class UserRepository {
  constructor() {
    this.model = User;
  }

  async create(data) {
    return await this.model.create(data);
  }

  async findById(id) {
    return await this.model.findById(id).populate('posts');
  }

  async findByEmail(email) {
    return await this.model.findOne({ email });
  }

  async update(id, data) {
    return await this.model.findByIdAndUpdate(id, data, {
      new: true,
      runValidators: true
    });
  }

  async delete(id) {
    return await this.model.findByIdAndDelete(id);
  }

  async findActiveUsers() {
    return await this.model.find({ isActive: true });
  }

  async getUserStats() {
    return await this.model.aggregate([
      { $group: { _id: '$role', count: { $sum: 1 } } }
    ]);
  }
}

module.exports = new UserRepository();

// Usage in controller
const userRepo = require('../repositories/userRepository');

app.get('/users/:id', async (req, res) => {
  const user = await userRepo.findById(req.params.id);
  res.json(user);
});
```

### **Service Layer Pattern:**
```javascript
// services/userService.js
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async createUser(userData) {
    // Business logic validation
    if (userData.age < 13) {
      throw new Error('Users must be at least 13 years old');
    }

    // Check if email already exists
    const existingUser = await this.userRepository.findByEmail(userData.email);
    if (existingUser) {
      throw new Error('Email address already exists');
    }

    // Hash password
    userData.password = await bcrypt.hash(userData.password, 12);

    // Create user
    const user = await this.userRepository.create(userData);

    // Send welcome email (fire and forget)
    this.sendWelcomeEmail(user.email);

    return user;
  }

  async authenticateUser(email, password) {
    const user = await this.userRepository.findByEmail(email);
    if (!user) {
      throw new Error('Invalid credentials');
    }

    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      throw new Error('Invalid credentials');
    }

    // Update last login
    await this.userRepository.update(user._id, {
      lastLogin: new Date(),
      loginCount: user.loginCount + 1
    });

    return user;
  }

  async sendWelcomeEmail(email) {
    // Implementation for sending welcome email
  }
}

module.exports = UserService;
```

### **Factory Pattern for Models:**
```javascript
// models/factory.js
class ModelFactory {
  static createUser(data) {
    const user = new User({
      name: data.name,
      email: data.email.toLowerCase(),
      age: data.age,
      role: data.role || 'user',
      isActive: true
    });

    return user;
  }

  static createPost(data, authorId) {
    const post = new Post({
      title: data.title,
      content: data.content,
      author: authorId,
      tags: data.tags || [],
      published: data.published || false
    });

    return post;
  }

  static createComment(data, authorId, postId) {
    const comment = new Comment({
      content: data.content,
      author: authorId,
      post: postId,
      isApproved: false
    });

    return comment;
  }
}

module.exports = ModelFactory;

// Usage
const user = ModelFactory.createUser(req.body);
await user.save();
```

### **Observer Pattern with Events:**
```javascript
// events/userEvents.js
const EventEmitter = require('events');

class UserEvents extends EventEmitter {
  constructor() {
    super();
    this.setupListeners();
  }

  setupListeners() {
    // Listen to model events and emit custom events
    User.watch().on('change', (change) => {
      switch (change.operationType) {
        case 'insert':
          this.emit('userCreated', change.fullDocument);
          break;
        case 'update':
          this.emit('userUpdated', change.documentKey._id, change.updateDescription);
          break;
        case 'delete':
          this.emit('userDeleted', change.documentKey._id);
          break;
      }
    });
  }

  onUserCreated(callback) {
    this.on('userCreated', callback);
  }

  onUserUpdated(callback) {
    this.on('userUpdated', callback);
  }

  onUserDeleted(callback) {
    this.on('userDeleted', callback);
  }
}

module.exports = new UserEvents();

// Usage
const userEvents = require('./events/userEvents');

userEvents.onUserCreated((user) => {
  console.log('New user created:', user.name);
  // Send welcome email
  // Update analytics
  // Send notification to admin
});

userEvents.onUserDeleted((userId) => {
  console.log('User deleted:', userId);
  // Clean up related data
  // Update analytics
});
```

### **Strategy Pattern for Queries:**
```javascript
// strategies/queryStrategies.js
class QueryStrategy {
  static async execute(query, options = {}) {
    const strategy = this.getStrategy(options.type || 'default');
    return await strategy(query, options);
  }

  static getStrategy(type) {
    const strategies = {
      default: this.defaultQuery,
      paginated: this.paginatedQuery,
      filtered: this.filteredQuery,
      search: this.searchQuery
    };

    return strategies[type] || strategies.default;
  }

  static async defaultQuery(query, options) {
    return await User.find(query);
  }

  static async paginatedQuery(query, options) {
    const { page = 1, limit = 10 } = options;
    const skip = (page - 1) * limit;

    const [docs, total] = await Promise.all([
      User.find(query).skip(skip).limit(limit),
      User.countDocuments(query)
    ]);

    return {
      docs,
      total,
      page,
      pages: Math.ceil(total / limit),
      hasNext: page * limit < total,
      hasPrev: page > 1
    };
  }

  static async filteredQuery(query, options) {
    const { filters = {} } = options;

    // Apply filters
    if (filters.age) {
      query.age = { $gte: filters.age.min, $lte: filters.age.max };
    }

    if (filters.role) {
      query.role = filters.role;
    }

    if (filters.isActive !== undefined) {
      query.isActive = filters.isActive;
    }

    return await User.find(query);
  }

  static async searchQuery(query, options) {
    const { search } = options;

    if (search) {
      query.$text = { $search: search };
    }

    return await User.find(query, { score: { $meta: 'textScore' } })
      .sort({ score: { $meta: 'textScore' } });
  }
}

module.exports = QueryStrategy;

// Usage
const result = await QueryStrategy.execute(
  { isActive: true },
  { type: 'paginated', page: 1, limit: 10 }
);
```

---

## 🔧 **Troubleshooting**

### **Common Issues and Solutions:**

#### **1. Connection Issues:**
```javascript
// Issue: Connection timeout
// Solution: Increase timeout settings
const options = {
  serverSelectionTimeoutMS: 5000, // Keep trying to send operations for 5 seconds
  socketTimeoutMS: 45000,         // Close sockets after 45 seconds of inactivity
  connectTimeoutMS: 10000         // Connection timeout
};

// Issue: Too many connections
// Solution: Configure connection pool
const options = {
  maxPoolSize: 10,      // Maximum number of connections in pool
  minPoolSize: 5,       // Minimum number of connections in pool
  maxIdleTimeMS: 30000  // Close connections after 30 seconds of inactivity
};
```

#### **2. Validation Issues:**
```javascript
// Issue: Validation errors not clear
// Solution: Custom error messages
userSchema.path('email').validate({
  validator: function(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  },
  message: 'Invalid email format'
});

// Issue: Required field validation not working
// Solution: Check field is properly defined
const userSchema = new mongoose.Schema({
  name: { type: String, required: true }, // ✅ Correct
  email: String // ❌ Missing required validation
});
```

#### **3. Performance Issues:**
```javascript
// Issue: Slow queries
// Solution: Add proper indexes
userSchema.index({ email: 1 });              // Index for email queries
userSchema.index({ name: 1, age: -1 });      // Compound index

// Issue: Memory issues
// Solution: Use lean queries for read-only operations
const users = await User.find().lean(); // Returns plain objects

// Issue: Large result sets
// Solution: Use pagination
const users = await User.find()
  .skip((page - 1) * limit)
  .limit(limit);
```

#### **4. Population Issues:**
```javascript
// Issue: Population not working
// Solution: Check reference is correct
const postSchema = new mongoose.Schema({
  author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' } // ✅ Correct
  // author: mongoose.Schema.Types.ObjectId // ❌ Missing ref
});

// Issue: Deep population not working
// Solution: Use proper syntax
const posts = await Post.find()
  .populate({
    path: 'author',
    populate: {
      path: 'profile' // Correct nested population
    }
  });
```

#### **5. Middleware Issues:**
```javascript
// Issue: Middleware not running
// Solution: Check middleware registration order
// Register middleware BEFORE creating model
userSchema.pre('save', function(next) {
  console.log('Pre-save middleware running');
  next();
});

const User = mongoose.model('User', userSchema); // Register AFTER middleware

// Issue: Async middleware not working
// Solution: Use proper async/await
userSchema.pre('save', async function(next) {
  try {
    // Async operations
    await someAsyncOperation();
    next();
  } catch (error) {
    next(error);
  }
});
```

### **Debugging Techniques:**

#### **1. Enable Mongoose Debug Mode:**
```javascript
// Enable debug mode
mongoose.set('debug', true);

// Or with custom function
mongoose.set('debug', (collectionName, method, query, doc) => {
  console.log(`${collectionName}.${method}`, JSON.stringify(query), doc);
});
```

#### **2. Monitor Query Performance:**
```javascript
// Add timing to queries
const start = Date.now();
const users = await User.find({ age: { $gte: 18 } });
const duration = Date.now() - start;
console.log(`Query took ${duration}ms`);

// Use explain() for query analysis
const explanation = await User.find({ age: { $gte: 18 } }).explain();
console.log('Query explanation:', explanation);
```

#### **3. Connection Monitoring:**
```javascript
// Monitor connection events
mongoose.connection.on('connecting', () => {
  console.log('Connecting to MongoDB...');
});

mongoose.connection.on('connected', () => {
  console.log('Connected to MongoDB');
});

mongoose.connection.on('disconnecting', () => {
  console.log('Disconnecting from MongoDB...');
});

mongoose.connection.on('disconnected', () => {
  console.log('Disconnected from MongoDB');
});

mongoose.connection.on('reconnected', () => {
  console.log('Reconnected to MongoDB');
});

// Monitor connection pool
setInterval(() => {
  console.log('Connection pool stats:', mongoose.connection.db.serverStatus());
}, 30000);
```

#### **4. Memory Usage Monitoring:**
```javascript
// Monitor Mongoose internal cache
setInterval(() => {
  const collections = Object.keys(mongoose.connection.collections);
  console.log('Cached collections:', collections.length);

  collections.forEach(name => {
    const collection = mongoose.connection.collections[name];
    console.log(`${name}: ${collection.bufferedOps.length} buffered ops`);
  });
}, 10000);

// Force garbage collection (development only)
if (global.gc) {
  setInterval(() => {
    global.gc();
    console.log('Garbage collection completed');
  }, 60000);
}
```

---

## 🚀 **Migration Guide**

### **Migrating from Native MongoDB Driver:**

#### **1. Basic CRUD Migration:**
```javascript
// Native MongoDB
const { MongoClient } = require('mongodb');

const client = await MongoClient.connect('mongodb://localhost:27017');
const db = client.db('myapp');
const collection = db.collection('users');

// Insert
await collection.insertOne({ name: 'John', email: 'john@example.com' });

// Find
const users = await collection.find({ age: { $gte: 18 } }).toArray();

// Update
await collection.updateOne(
  { email: 'john@example.com' },
  { $set: { lastLogin: new Date() } }
);

// Delete
await collection.deleteOne({ email: 'john@example.com' });

// Mongoose equivalent
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  age: Number,
  lastLogin: Date
});

const User = mongoose.model('User', userSchema);

// Insert
await User.create({ name: 'John', email: 'john@example.com' });

// Find
const users = await User.find({ age: { $gte: 18 } });

// Update
await User.updateOne(
  { email: 'john@example.com' },
  { lastLogin: new Date() }
);

// Delete
await User.deleteOne({ email: 'john@example.com' });
```

#### **2. Schema Migration:**
```javascript
// Define schema with validations
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  age: {
    type: Number,
    min: 0,
    max: 120
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Add indexes
userSchema.index({ email: 1 });
userSchema.index({ createdAt: -1 });

// Add methods
userSchema.methods.getFullName = function() {
  return this.name;
};

userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email });
};
```

#### **3. Connection Migration:**
```javascript
// Native MongoDB connection
const client = new MongoClient('mongodb://localhost:27017', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

await client.connect();
const db = client.db('myapp');

// Mongoose connection
await mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Multiple databases
const db1 = mongoose.createConnection('mongodb://localhost:27017/db1');
const db2 = mongoose.createConnection('mongodb://localhost:27017/db2');
```

### **Best Practices for Migration:**

#### **1. Gradual Migration:**
```javascript
// Start with read operations
// Phase 1: Migrate read operations to Mongoose
const users = await User.find({}); // New Mongoose way
// const users = await collection.find({}).toArray(); // Old way

// Phase 2: Migrate write operations
await User.create(userData); // New way
// await collection.insertOne(userData); // Old way

// Phase 3: Add validations and middleware
// Add schema validations, indexes, middleware gradually

// Phase 4: Optimize and refactor
// Use Mongoose features like population, virtuals, etc.
```

#### **2. Data Validation Migration:**
```javascript
// Add validation gradually
const userSchema = new mongoose.Schema({
  name: String, // Start without validation
  email: String,
  age: Number
});

// Later add validations
userSchema.path('name').required(true);
userSchema.path('email').required(true).unique(true);
userSchema.path('age').min(0).max(120);

// Or redefine schema
const newUserSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  age: { type: Number, min: 0, max: 120 }
});
```

#### **3. Index Migration:**
```javascript
// Analyze existing indexes
const indexes = await db.collection('users').indexes();

// Create equivalent Mongoose indexes
userSchema.index({ email: 1 }, { unique: true });
userSchema.index({ name: 1 });
userSchema.index({ createdAt: -1 });

// For compound indexes
userSchema.index({ name: 1, age: -1 });
```

### **Common Migration Issues:**

#### **1. ObjectId Handling:**
```javascript
// Native MongoDB
const userId = new ObjectId('507f1f77bcf86cd799439011');

// Mongoose
const userId = mongoose.Types.ObjectId('507f1f77bcf86cd799439011');
// or
const userId = '507f1f77bcf86cd799439011'; // Mongoose auto-converts
```

#### **2. Date Handling:**
```javascript
// Native MongoDB stores dates as ISO strings or Date objects
// Mongoose automatically handles conversion

// When querying dates
const users = await User.find({
  createdAt: {
    $gte: new Date('2023-01-01'),
    $lt: new Date('2024-01-01')
  }
});
```

#### **3. Error Handling:**
```javascript
// Native MongoDB error handling
try {
  await collection.insertOne(userData);
} catch (error) {
  if (error.code === 11000) {
    console.log('Duplicate key error');
  }
}

// Mongoose error handling
try {
  await User.create(userData);
} catch (error) {
  if (error.code === 11000) {
    console.log('Duplicate key error');
  } else if (error.name === 'ValidationError') {
    console.log('Validation error:', error.errors);
  }
}
```

### **Performance Considerations:**

#### **1. Connection Pooling:**
```javascript
// Native MongoDB
const client = new MongoClient(uri, {
  maxPoolSize: 10,
  minPoolSize: 5
});

// Mongoose
const options = {
  maxPoolSize: 10,
  minPoolSize: 5,
  maxIdleTimeMS: 30000
};
```

#### **2. Query Optimization:**
```javascript
// Use lean queries for better performance
const users = await User.find().lean(); // Returns plain objects

// Use select to limit fields
const users = await User.find().select('name email');

// Use indexes effectively
userSchema.index({ email: 1 });
userSchema.index({ createdAt: -1 });
```

### **Testing Migration:**
```javascript
// Test script for migration validation
async function testMigration() {
  try {
    // Test basic CRUD operations
    const user = await User.create({
      name: 'Test User',
      email: 'test@example.com',
      age: 25
    });

    const foundUser = await User.findById(user._id);
    assert(foundUser.name === 'Test User');

    const updatedUser = await User.findByIdAndUpdate(
      user._id,
      { age: 26 },
      { new: true }
    );
    assert(updatedUser.age === 26);

    await User.findByIdAndDelete(user._id);

    console.log('✅ Migration tests passed');
  } catch (error) {
    console.error('❌ Migration test failed:', error);
  }
}

testMigration();
```

---

## 🎉 **Conclusion**

Mongoose is a powerful and flexible ODM that brings structure, validation, and productivity to MongoDB development. This comprehensive guide covers everything from basic concepts to advanced features, providing you with the knowledge needed to build robust, scalable applications.

**Key Takeaways:**
- ✅ **Schema Definition** provides structure and validation
- ✅ **Models and Documents** offer high-level abstractions
- ✅ **Middleware** enables powerful pre/post-processing
- ✅ **Validation** ensures data integrity
- ✅ **Population** handles relationships efficiently
- ✅ **Plugins** extend functionality
- ✅ **Best Practices** ensure maintainable code
- ✅ **Performance Optimization** keeps applications fast

**When to Use Mongoose:**
- Building Node.js applications with MongoDB
- Needing data validation and structure
- Working with complex relationships
- Requiring middleware and plugins
- Building scalable applications
- Needing type safety and better error handling

**Migration Path:**
1. Start with simple read operations
2. Gradually add write operations
3. Implement validations and middleware
4. Add indexes and optimize queries
5. Leverage advanced features like population and plugins

This guide should serve as your comprehensive reference for Mongoose development, helping you build better, more maintainable MongoDB applications with Node.js.
