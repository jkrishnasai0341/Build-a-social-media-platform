# Build-a-social-media-platform
✅ Tech Stack:
Frontend: HTML + Bootstrap
Backend: Node.js + Express
Database: MongoDB
Authentication: Express-session with bcrypt
Validation: Express-validator

🔧 Step-by-Step Breakdown

1. Project Structure
employee-management/
├── models/
│   └── Employee.js
├── routes/
│   ├── auth.js
│   └── employee.js
├── views/
│   ├── login.ejs
│   ├── dashboard.ejs
│   └── employees.ejs
├── public/
│   └── css/
├── app.js
├── package.json
└── .env

2. 📦 package.json Dependencies
Install them using:
npm init -y
npm install express mongoose express-session bcryptjs express-validator dotenv ejs connect-mongo

3. 📁 models/Employee.js
const mongoose = require('mongoose');

const employeeSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, unique: true, required: true },
  phone: String,
  department: String,
  jobTitle: String,
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Employee', employeeSchema);

4. 📁 routes/auth.js
const express = require('express');
const bcrypt = require('bcryptjs');
const router = express.Router();

// Dummy user for login
const adminUser = {
  username: 'admin',
  passwordHash: bcrypt.hashSync('admin123', 10)
};

router.get('/login', (req, res) => {
  res.render('login');
});

router.post('/login', async (req, res) => {
  const { username, password } = req.body;
  if (username === adminUser.username && bcrypt.compareSync(password, adminUser.passwordHash)) {
    req.session.isAuth = true;
    return res.redirect('/dashboard');
  }
  res.render('login', { error: 'Invalid credentials' });
});

router.get('/logout', (req, res) => {
  req.session.destroy(err => {
    res.redirect('/login');
  });
});

module.exports = router;

5. 📁 routes/employee.js
const express = require('express');
const { body, validationResult } = require('express-validator');
const Employee = require('../models/Employee');
const router = express.Router();

const isAuth = (req, res, next) => {
  if (req.session.isAuth) return next();
  res.redirect('/login');
};

// Read
router.get('/', isAuth, async (req, res) => {
  const employees = await Employee.find();
  res.render('employees', { employees });
});

// Create
router.post('/add',
  isAuth,
  body('email').isEmail().withMessage('Invalid email'),
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).send('Validation failed');
    const { name, email, phone, department, jobTitle } = req.body;
    const employee = new Employee({ name, email, phone, department, jobTitle });
    await employee.save();
    res.redirect('/employees');
});

// Update
router.post('/update/:id', isAuth, async (req, res) => {
  const { name, email, phone, department, jobTitle } = req.body;
  await Employee.findByIdAndUpdate(req.params.id, { name, email, phone, department, jobTitle });
  res.redirect('/employees');
});

// Delete
router.post('/delete/:id', isAuth, async (req, res) => {
  await Employee.findByIdAndDelete(req.params.id);
  res.redirect('/employees');
});

module.exports = router;

6. 📁 views/login.ejs
<!DOCTYPE html>
<html>
<head><title>Login</title></head>
<body>
  <form method="POST" action="/login">
    <h2>Admin Login</h2>
    <input name="username" placeholder="Username" required>
    <input name="password" type="password" placeholder="Password" required>
    <button type="submit">Login</button>
    <% if (typeof error !== 'undefined') { %><p><%= error %></p><% } %>
  </form>
</body>
</html>

7. 📁 views/employees.ejs
<!DOCTYPE html>
<html>
<head><title>Employee List</title></head>
<body>
  <h2>Employee Records</h2>
  <form method="POST" action="/employees/add">
    <input name="name" placeholder="Name">
    <input name="email" placeholder="Email">
    <button type="submit">Add</button>
  </form>
  <ul>
    <% employees.forEach(emp => { %>
      <li><%= emp.name %> - <%= emp.email %></li>
    <% }); %>
  </ul>
  <a href="/logout">Logout</a>
</body>
</html>
8. 📄 app.js
const express = require('express');
const session = require('express-session');
const MongoStore = require('connect-mongo');
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const authRoutes = require('./routes/auth');
const employeeRoutes = require('./routes/employee');

dotenv.config();
const app = express();
app.set('view engine', 'ejs');
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true });

app.use(session({
  secret: 'secret123',
  resave: false,
  saveUninitialized: false,
  store: MongoStore.create({ mongoUrl: process.env.MONGO_URI }),
  cookie: { maxAge: 1000 * 60 * 60 }
}));

app.use(authRoutes);
app.use('/employees', employeeRoutes);
app.get('/dashboard', (req, res) => res.redirect('/employees'));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(Server running on port ${PORT}));

9. 🔐 .env
MONGO_URI=mongodb://localhost:27017/employee_db

✅ Final Notes:
Launch MongoDB locally or use MongoDB Atlas.
Start server with node app.js
Access at: http://localhost:3000/login (login with admin / admin123)
