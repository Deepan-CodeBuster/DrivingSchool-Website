SRI DEVI DRIVING SCHOOL
https://sridevidrivingschool.vercel.app/


Here's a **complete working example** of a MERN stack application that includes a form for uploading:

* **Name (text)**
* **Price (number)**
* **Image (file)**

Uploaded images are stored on **Cloudinary**, and form data is stored in **MongoDB**.

---

## 🛠 Stack Used:

* **MongoDB**
* **Express**
* **React**
* **Node.js**
* **Cloudinary** for image storage
* **Multer** for parsing multipart/form-data
* **Axios** for HTTP requests

---

# 🧠 Project Structure

```
client/
├── src/
│   ├── App.js
│   └── UploadForm.js

server/
├── models/
│   └── Product.js
├── routes/
│   └── upload.js
├── .env
├── server.js
```

---

## 🔧 Backend (Node.js + Express)

### Step 1: Install dependencies

In `server/`:

```bash
npm init -y
npm install express mongoose multer cloudinary dotenv cors
```

### Step 2: `.env` file

```env
MONGO_URI=your_mongodb_connection_string
CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret
PORT=5000
```

### Step 3: `models/Product.js`

```js
const mongoose = require('mongoose');

const ProductSchema = new mongoose.Schema({
  name: String,
  price: Number,
  imageUrl: String,
});

module.exports = mongoose.model('Product', ProductSchema);
```

### Step 4: `routes/upload.js`

```js
const express = require('express');
const router = express.Router();
const multer = require('multer');
const Product = require('../models/Product');
const cloudinary = require('cloudinary').v2;
const fs = require('fs');
require('dotenv').config();

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
});

const upload = multer({ dest: 'uploads/' });

router.post('/', upload.single('image'), async (req, res) => {
  const { name, price } = req.body;
  const imagePath = req.file.path;

  try {
    const result = await cloudinary.uploader.upload(imagePath, {
      folder: 'products',
    });

    const product = new Product({
      name,
      price,
      imageUrl: result.secure_url,
    });

    await product.save();
    fs.unlinkSync(imagePath); // delete temp file

    res.status(200).json({ message: 'Upload successful', product });
  } catch (err) {
    res.status(500).json({ error: 'Upload failed', details: err });
  }
});

module.exports = router;
```

### Step 5: `server.js`

```js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();
const uploadRoute = require('./routes/upload');

app.use(cors());
app.use(express.json());
app.use('/api/upload', uploadRoute);

mongoose.connect(process.env.MONGO_URI)
  .then(() => {
    console.log('MongoDB connected');
    app.listen(process.env.PORT || 5000, () =>
      console.log('Server running on port', process.env.PORT)
    );
  })
  .catch((err) => console.error(err));
```

---

## 🌐 Frontend (React)

### Step 1: Create React App

```bash
npx create-react-app client
cd client
npm install axios
```

### Step 2: `src/UploadForm.js`

```jsx
import React, { useState } from 'react';
import axios from 'axios';

function UploadForm() {
  const [formData, setFormData] = useState({
    name: '',
    price: '',
    image: null,
  });

  const handleChange = (e) => {
    const { name, value, files } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]: files ? files[0] : value,
    }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    const data = new FormData();
    data.append('name', formData.name);
    data.append('price', formData.price);
    data.append('image', formData.image);

    try {
      const res = await axios.post('http://localhost:5000/api/upload', data);
      alert('Upload successful!');
      console.log(res.data);
    } catch (err) {
      console.error(err);
      alert('Upload failed.');
    }
  };

  return (
    <form onSubmit={handleSubmit} encType="multipart/form-data">
      <div>
        <label>Name: </label>
        <input name="name" type="text" onChange={handleChange} required />
      </div>
      <div>
        <label>Price (Rs): </label>
        <input name="price" type="number" onChange={handleChange} required />
      </div>
      <div>
        <label>Image: </label>
        <input name="image" type="file" accept="image/*" onChange={handleChange} required />
      </div>
      <button type="submit">Upload</button>
    </form>
  );
}

export default UploadForm;
```

### Step 3: `src/App.js`

```jsx
import React from 'react';
import UploadForm from './UploadForm';

function App() {
  return (
    <div className="App">
      <h2>Upload Product</h2>
      <UploadForm />
    </div>
  );
}

export default App;
```

---

## ✅ Final Notes

* Start the **server**:

```bash
cd server
node server.js
```

* Start the **client**:

```bash
cd client
npm start
```

---

## 🧪 Test It

Visit: [http://localhost:3000](http://localhost:3000)

Try uploading a product with name, price, and image. Image goes to **Cloudinary**, and data is saved in **MongoDB**.

---

Would you like me to bundle this into a downloadable ZIP or GitHub repo for you?




To **fetch and display** the uploaded product details (Name, Price, Image URL) from MongoDB in a new component called `Display.jsx`, here's how you can do it.

This assumes:

* You have already set up your backend route to store product data.
* Your backend exposes an API endpoint like `GET /api/upload` to fetch products.

---

## ✅ Step 1: Backend – Add Route to Fetch All Products

In your `routes/upload.js`, add this at the bottom:

```js
// GET all products
router.get('/', async (req, res) => {
  try {
    const products = await Product.find().sort({ _id: -1 });
    res.status(200).json(products);
  } catch (err) {
    res.status(500).json({ error: 'Failed to fetch products' });
  }
});
```

This lets you fetch all products from `GET /api/upload`.

---

## ✅ Step 2: Create `Display.jsx` in React

Create a new file: `client/src/Display.jsx`

```jsx
import React, { useEffect, useState } from 'react';
import axios from 'axios';

function Display() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    axios.get('http://localhost:5000/api/upload')
      .then((res) => setProducts(res.data))
      .catch((err) => console.error('Failed to fetch products:', err));
  }, []);

  return (
    <div>
      <h2>Uploaded Products</h2>
      {products.length === 0 ? (
        <p>No products uploaded yet.</p>
      ) : (
        <div style={{ display: 'flex', flexWrap: 'wrap', gap: '1rem' }}>
          {products.map((product) => (
            <div key={product._id} style={{
              border: '1px solid #ccc',
              borderRadius: '8px',
              padding: '1rem',
              width: '200px',
              textAlign: 'center'
            }}>
              <img
                src={product.imageUrl}
                alt={product.name}
                style={{ width: '100%', height: 'auto', borderRadius: '4px' }}
              />
              <h4>{product.name}</h4>
              <p>Price: ₹{product.price}</p>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

export default Display;
```

---

## ✅ Step 3: Update `App.js` to Show Both Upload and Display

Edit `client/src/App.js`:

```jsx
import React from 'react';
import UploadForm from './UploadForm';
import Display from './Display';

function App() {
  return (
    <div style={{ padding: '2rem' }}>
      <h1>Product Manager</h1>
      <UploadForm />
      <hr style={{ margin: '2rem 0' }} />
      <Display />
    </div>
  );
}

export default App;
```

---

## 🧪 Result

When you open the app:

* You can upload a new product.
* Below the form, all uploaded products will be shown with:

  * Image (from Cloudinary)
  * Name
  * Price

---

## ✅ Summary

* ✅ `POST /api/upload` to upload a new product.
* ✅ `GET /api/upload` to fetch all products.
* ✅ `Display.jsx` shows all products.
* ✅ Fully integrated with MongoDB and Cloudinary.



```bash
import cv2

face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')

img = cv2.imread('image.png')

gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

faces = face_cascade.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5)

for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x + w, y + h), (255, 0, 0), 2)

cv2.imshow('img', img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```



---

Let me know if you'd like pagination, search, or edit/delete features added!
