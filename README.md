# Build-a-social-media-platform
✅ Core Features
User Registration/Login
User Profiles with bio, profile picture
Posts (text + media uploads)
Like & Comment functionality
Feed UI showing latest posts
Media Upload (images/videos)
Tagging users in posts
🔧 Optional Features
Follow/unfollow system
Real-time notifications
Explore/trending page
Save/bookmark posts
Direct messaging

🛠️ Suggested Tech Stack
Component
Technology

Frontend
React.js + Tailwind CSS or Bootstrap

Backend
Node.js + Express.js

Database
MongoDB (with Mongoose)

Authentication
JWT + bcrypt

Media Storage
Cloudinary / Firebase Storage

Real-Time
Socket.IO (for notifications, optional)


🗂️ Folder Structure
social-platform/
├── client/           # React frontend
│   └── src/
│       ├── components/
│       ├── pages/
│       ├── api/
│       └── App.js
├── server/           # Express backend
│   ├── models/
│   ├── routes/
│   ├── controllers/
│   ├── middleware/
│   └── server.js

🧠 Step-by-Step Code Guide

1. 🚀 Backend Setup
1.1 server/server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const authRoutes = require('./routes/authRoutes');
const postRoutes = require('./routes/postRoutes');

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect('mongodb://localhost:27017/social-platform')
    .then(() => console.log("DB connected"));

app.use('/api/auth', authRoutes);
app.use('/api/posts', postRoutes);

app.listen(5000, () => console.log("Server running on port 5000"));

1.2 Models
models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  username: { type: String, unique: true },
  email: String,
  password: String,
  bio: String,
  profilePic: String
});

module.exports = mongoose.model('User', userSchema);
models/Post.js
const mongoose = require('mongoose');

const postSchema = new mongoose.Schema({
  userId: mongoose.Schema.Types.ObjectId,
  content: String,
  imageUrl: String,
  likes: [mongoose.Schema.Types.ObjectId],
  comments: [{
    userId: mongoose.Schema.Types.ObjectId,
    text: String,
    createdAt: { type: Date, default: Date.now }
  }],
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Post', postSchema);

1.3 Routes
routes/authRoutes.js
const express = require('express');
const router = express.Router();
const User = require('../models/User');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

// Register
router.post('/register', async (req, res) => {
  const { username, email, password } = req.body;
  const hashed = await bcrypt.hash(password, 10);
  const user = new User({ username, email, password: hashed });
  await user.save();
  res.send("User registered");
});

// Login
router.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = await User.findOne({ username });
  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) return res.status(400).send("Invalid credentials");
  const token = jwt.sign({ id: user._id }, 'secret');
  res.json({ token });
});

module.exports = router;

routes/postRoutes.js
const express = require('express');
const router = express.Router();
const Post = require('../models/Post');

// Create post
router.post('/', async (req, res) => {
  const post = new Post(req.body);
  await post.save();
  res.json(post);
});

// Get all posts
router.get('/', async (req, res) => {
  const posts = await Post.find().sort({ createdAt: -1 });
  res.json(posts);
});

// Like post
router.put('/like/:id', async (req, res) => {
  await Post.findByIdAndUpdate(req.params.id, {
    $addToSet: { likes: req.body.userId }
  });
  res.send("Liked");
});

module.exports = router;

2. 🖼️ Frontend (React)
2.1 App.js (simple post feed and creator)
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [content, setContent] = useState('');
  const [posts, setPosts] = useState([]);
  const createPost = async () => {
    await axios.post('http://localhost:5000/api/posts', {
      content,
      userId: "123", // replace with actual auth
      imageUrl: ""
    });
    setContent('');
    fetchPosts();
  };

  const fetchPosts = async () => {
    const res = await axios.get('http://localhost:5000/api/posts');
    setPosts(res.data);
  };

  useEffect(() => {
    fetchPosts();
  }, []);

  return (
    <div>
      <h1>Social Feed</h1>
      <textarea value={content} onChange={(e) => setContent(e.target.value)} />
      <button onClick={createPost}>Post</button>

      {posts.map(post => (
        <div key={post._id}>
          <p>{post.content}</p>
          <small>{new Date(post.createdAt).toLocaleString()}</small>
        </div>
      ))}
    </div>
  );
}

export default App;

✅ Optional Feature Extensions
Feature
How to Implement

Follow system
Add followers and following arrays in User model

Real-time notifications
Use Socket.IO to emit events on new likes/comments

File uploads
Use Multer or Cloudinary for imageUrl

Explore page
Sort posts by likes or tags

Tagging users
Parse @username mentions and link to profiles
