// ============================================================
// WEEK 6: TESTING AND DEBUGGING IN MERN APPLICATIONS
// PROJECT: BUG TRACKER (Single Code Sheet)
// ============================================================

// ============================================================
// 📁 BACKEND SETUP (Express + MongoDB + Mongoose)
// ============================================================
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const app = express();
const PORT = 5000;

// Middleware
app.use(cors());
app.use(express.json());

// MongoDB Connection
mongoose
  .connect('mongodb://localhost:27017/mern-bug-tracker')
  .then(() => console.log('MongoDB connected'))
  .catch((err) => console.error('Mongo error', err));

// Bug Schema
const bugSchema = new mongoose.Schema({
  title: String,
  description: String,
  status: { type: String, default: 'open' },
});

const Bug = mongoose.model('Bug', bugSchema);

// ============================================================
// 📁 API ROUTES (CRUD)
// ============================================================
app.post('/api/bugs', async (req, res, next) => {
  try {
    const bug = new Bug(req.body);
    await bug.save();
    res.status(201).json(bug);
  } catch (err) {
    next(err);
  }
});

app.get('/api/bugs', async (req, res, next) => {
  try {
    const bugs = await Bug.find();
    res.json(bugs);
  } catch (err) {
    next(err);
  }
});

app.put('/api/bugs/:id', async (req, res, next) => {
  try {
    const bug = await Bug.findByIdAndUpdate(req.params.id, req.body, { new: true });
    res.json(bug);
  } catch (err) {
    next(err);
  }
});

app.delete('/api/bugs/:id', async (req, res, next) => {
  try {
    await Bug.findByIdAndDelete(req.params.id);
    res.json({ message: 'Bug deleted' });
  } catch (err) {
    next(err);
  }
});

// ============================================================
// 📁 ERROR HANDLING MIDDLEWARE
// ============================================================
app.use((err, req, res, next) => {
  console.error('🔥 Error:', err.message);
  res.status(500).json({ error: 'Something went wrong' });
});

// ============================================================
// 📁 START SERVER
// ============================================================
app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));

// ============================================================
// 📁 FRONTEND (REACT APP - App.jsx)
// ============================================================
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [bugs, setBugs] = useState([]);
  const [form, setForm] = useState({ title: '', description: '' });
  const [error, setError] = useState('');

  const loadBugs = async () => {
    try {
      const res = await axios.get('http://localhost:5000/api/bugs');
      setBugs(res.data);
    } catch (e) {
      setError('Failed to load bugs');
    }
  };

  useEffect(() => {
    loadBugs();
  }, []);

  const submitBug = async () => {
    try {
      await axios.post('http://localhost:5000/api/bugs', form);
      setForm({ title: '', description: '' });
      loadBugs();
    } catch (e) {
      setError('Submission failed');
    }
  };

  const updateStatus = async (id, status) => {
    try {
      await axios.put(`http://localhost:5000/api/bugs/${id}`, { status });
      loadBugs();
    } catch (e) {
      setError('Update failed');
    }
  };

  const deleteBug = async (id) => {
    try {
      await axios.delete(`http://localhost:5000/api/bugs/${id}`);
      loadBugs();
    } catch (e) {
      setError('Delete failed');
    }
  };

  return (
    <div>
      <h1>🐞 Bug Tracker</h1>
      {error && <p style={{ color: 'red' }}>{error}</p>}
      <input
        placeholder="Bug Title"
        value={form.title}
        onChange={(e) => setForm({ ...form, title: e.target.value })}
      />
      <input
        placeholder="Description"
        value={form.description}
        onChange={(e) => setForm({ ...form, description: e.target.value })}
      />
      <button onClick={submitBug}>Submit</button>
      <ul>
        {bugs.map((bug) => (
          <li key={bug._id}>
            <b>{bug.title}</b>: {bug.description} — {bug.status}
            <button onClick={() => updateStatus(bug._id, 'in-progress')}>In Progress</button>
            <button onClick={() => updateStatus(bug._id, 'resolved')}>Resolved</button>
            <button onClick={() => deleteBug(bug._id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;

// ============================================================
// 📁 TESTING (USING JEST & SUPERTEST - backend)
// ============================================================

/*
// Install: npm install --save-dev jest supertest
// In package.json add: "test": "jest"
*/

const request = require('supertest');
const expressApp = require('./index'); // if split into file
const Bug = require('./models/Bug'); // if split

describe('Bug API', () => {
  it('should create a new bug', async () => {
    const res = await request(expressApp)
      .post('/api/bugs')
      .send({ title: 'Bug 1', description: 'Something is broken' });
    expect(res.statusCode).toEqual(201);
    expect(res.body.title).toBe('Bug 1');
  });
});

// ============================================================
// 📁 FRONTEND TESTING (React Testing Library)
// ============================================================

/*
// Install: npm install --save-dev @testing-library/react jest
*/

import { render, fireEvent, screen } from '@testing-library/react';
import App from './App';

test('renders bug tracker title', () => {
  render(<App />);
  const linkElement = screen.getByText(/Bug Tracker/i);
  expect(linkElement).toBeInTheDocument();
});

// ============================================================
// 📁 DEBUGGING (INTENTIONAL BUG + TOOLS)
// ============================================================

// INTENTIONAL BUG: Misspelled 'status' in API
// await Bug.findByIdAndUpdate(req.params.id, { statuss: req.body.status })

// DEBUGGING TOOLS:
// - console.log(req.body) // Track API input
// - Chrome DevTools → Network tab to inspect requests
// - node --inspect index.js → for Node.js debugger
// - React Developer Tools for component state
// - Add <ErrorBoundary> in React for catching crashes

// Example Error Boundary:
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  componentDidCatch(error, info) {
    console.log('Boundary caught:', error);
  }
  render() {
    if (this.state.hasError) return <h2>Something went wrong.</h2>;
    return this.props.children;
  }
}

// Wrap <App /> in ErrorBoundary:
// <ErrorBoundary><App /></ErrorBoundary>

// ============================================================
// 📁 README.md CONTENT (for your GitHub)
// ============================================================

/*

# MERN Bug Tracker 🐞

## Installation
```bash
git clone https://github.com/yourname/mern-bug-tracker
cd mern-bug-tracker
npm install && cd client && npm install
