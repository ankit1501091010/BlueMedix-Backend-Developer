# BlueMedix-Backend-Developer

# User Model:

// models/User.js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  roles: {
    isSeller: { type: Boolean, default: false },
    isCustomer: { type: Boolean, default: true },
    isAdmin: { type: Boolean, default: false },
  },
}, { timestamps: true });

// Hash password before saving user
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 10);
  next();
});

// Method to compare password
userSchema.methods.comparePassword = function(password) {
  return bcrypt.compare(password, this.password);
};

module.exports = mongoose.model('User', userSchema);



# Input Validation:

// validation/userValidation.js
const Joi = require('joi');

const userValidationSchema = Joi.object({
  username: Joi.string().min(3).max(30).required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(6).required(),
  roles: Joi.object({
    isSeller: Joi.boolean(),
    isCustomer: Joi.boolean(),
    isAdmin: Joi.boolean(),
  }),
});

const updateUserValidationSchema = Joi.object({
  username: Joi.string().min(3).max(30),
  email: Joi.string().email(),
  roles: Joi.object({
    isSeller: Joi.boolean(),
    isCustomer: Joi.boolean(),
    isAdmin: Joi.boolean(),
  }),
});

module.exports = {
  userValidationSchema,
  updateUserValidationSchema,
};


# User Controller:
// controllers/userController.js
const User = require('../models/User');
const { userValidationSchema, updateUserValidationSchema } = require('../validation/userValidation');

// Create User
exports.createUser = async (req, res) => {
  const { error } = userValidationSchema.validate(req.body);
  if (error) return res.status(400).json({ error: error.details[0].message });

  try {
    const { username, email, password, roles } = req.body;
    const user = new User({ username, email, password, roles });
    await user.save();
    res.status(201).json({ message: 'User created successfully', user });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};

// Get User Details
exports.getUserDetails = async (req, res) => {
  const { userId } = req.params;
  
  try {
    const user = await User.findById(userId);
    if (!user) return res.status(404).json({ message: 'User not found' });
    res.status(200).json(user);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};

// Update User Information
exports.updateUser = async (req, res) => {
  const { error } = updateUserValidationSchema.validate(req.body);
  if (error) return res.status(400).json({ error: error.details[0].message });

  const { userId } = req.params;
  try {
    const user = await User.findByIdAndUpdate(userId, req.body, { new: true });
    if (!user) return res.status(404).json({ message: 'User not found' });
    res.status(200).json({ message: 'User updated successfully', user });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};

// Delete User
exports.deleteUser = async (req, res) => {
  const { userId } = req.params;

  try {
    const user = await User.findByIdAndDelete(userId);
    if (!user) return res.status(404).json({ message: 'User not found' });
    res.status(200).json({ message: 'User deleted successfully' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};


#  Routes:
// routes/userRoutes.js
const express = require('express');
const { createUser, getUserDetails, updateUser, deleteUser } = require('../controllers/userController');
const router = express.Router();

// Create user
router.post('/users', createUser);

// Get user details
router.get('/users/:userId', getUserDetails);

// Update user details
router.put('/users/:userId', updateUser);

// Delete user
router.delete('/users/:userId', deleteUser);

module.exports = router;
# Set Up the Express App:

// app.js
const express = require('express');
const mongoose = require('mongoose');
const userRoutes = require('./routes/userRoutes');
const dotenv = require('dotenv');
dotenv.config();

const app = express();

// Middleware
app.use(express.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch((err) => console.log('MongoDB connection error:', err));

// Routes
app.use('/api', userRoutes);

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));


#  Testing the API
# 1. Create User (POST /api/users)
{
  "username": "ankit",
  "email": "ankit@example.com",
  "password": "securepassword",
  "roles": {
    "isSeller": true,
    "isCustomer": true,
    "isAdmin": false
  }
}

# 2. Get User Details (GET /api/users/:userId)

# 3. Update User Information (PUT /api/users/:userId)

Request body:
 {
  "username": "ankit_updated",
  "roles": {
    "isSeller": true,
    "isCustomer": false,
    "isAdmin": true
  }
}

# 4. Delete User (DELETE /api/users/:userId)

