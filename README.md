# desinext-app
import os
import zipfile

# Set up directory structure
base_dir = "/mnt/data/desinext-app"
folders = [
    "public",
    "src/assets",
    "src/components",
    "src/pages",
    "src/utils"
]

files = {
    "public/index.html": """<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>DesiNext App</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>""",

    "src/index.js": """import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
""",

    "src/index.css": """@tailwind base;
@tailwind components;
@tailwind utilities;""",

    "src/App.jsx": """import { BrowserRouter as Router, Routes, Route } from "react-router-dom";
import Splash from "./pages/Splash";
import Home from "./pages/Home";
import Marketplace from "./pages/Marketplace";
import Cart from "./pages/Cart";
import Wishlist from "./pages/Wishlist";
import NavDrawer from "./components/NavDrawer";

export default function App() {
  return (
    <Router>
      <NavDrawer />
      <Routes>
        <Route path="/" element={<Splash />} />
        <Route path="/home" element={<Home />} />
        <Route path="/marketplace" element={<Marketplace />} />
        <Route path="/cart" element={<Cart />} />
        <Route path="/wishlist" element={<Wishlist />} />
      </Routes>
    </Router>
  );
}""",

    "src/pages/Splash.jsx": """import { motion } from "framer-motion";
import { useEffect } from "react";
import { useNavigate } from "react-router-dom";

export default function Splash() {
  const navigate = useNavigate();
  useEffect(() => {
    setTimeout(() => {
      navigate("/home");
    }, 3000);
  }, []);

  return (
    <div className="h-screen flex items-center justify-center bg-maroon-700 text-white">
      <motion.h1
        className="text-4xl font-bold"
        initial={{ scale: 0 }}
        animate={{ scale: 1 }}
        transition={{ duration: 1 }}
      >
        DesiNext
      </motion.h1>
    </div>
  );
}""",

    "src/pages/Home.jsx": """import VoiceAssistant from "../components/VoiceAssistant";

export default function Home() {
  return (
    <div className="mt-16 p-4">
      <h1 className="text-2xl font-semibold">Welcome to DesiNext</h1>
      <VoiceAssistant />
    </div>
  );
}""",

    "src/pages/Marketplace.jsx": """import { useState, useEffect } from "react";
import ProductCard from "../components/ProductCard";

export default function Marketplace() {
  const [products, setProducts] = useState([]);
  const [location, setLocation] = useState("");

  useEffect(() => {
    setProducts([
      { id: 1, name: "Handloom Saree", price: 1200 },
      { id: 2, name: "Terracotta Pot", price: 400 },
    ]);

    navigator.geolocation.getCurrentPosition(async (pos) => {
      const { latitude, longitude } = pos.coords;
      const response = await fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${latitude}&lon=${longitude}`);
      const data = await response.json();
      setLocation(data.address.city || data.display_name);
    });
  }, []);

  return (
    <div className="mt-16 p-4">
      <h2 className="text-xl mb-2">Marketplace - {location}</h2>
      <div className="grid grid-cols-2 gap-4">
        {products.map((p) => (
          <ProductCard key={p.id} product={p} />
        ))}
      </div>
    </div>
  );
}""",

    "src/pages/Cart.jsx": """export default function Cart() {
  const cart = JSON.parse(localStorage.getItem("cart") || "[]");

  return (
    <div className="mt-16 p-4">
      <h2 className="text-xl font-semibold mb-2">Your Cart</h2>
      {cart.length === 0 ? <p>No items in cart.</p> : (
        <ul className="space-y-2">
          {cart.map((item, index) => (
            <li key={index} className="border p-2 rounded">{item.name} - ₹{item.price}</li>
          ))}
        </ul>
      )}
    </div>
  );
}""",

    "src/pages/Wishlist.jsx": """export default function Wishlist() {
  const wishlist = JSON.parse(localStorage.getItem("wishlist") || "[]");

  return (
    <div className="mt-16 p-4">
      <h2 className="text-xl font-semibold mb-2">Your Wishlist</h2>
      {wishlist.length === 0 ? <p>No items in wishlist.</p> : (
        <ul className="space-y-2">
          {wishlist.map((item, index) => (
            <li key={index} className="border p-2 rounded">{item.name} - ₹{item.price}</li>
          ))}
        </ul>
      )}
    </div>
  );
}""",

    "src/components/NavDrawer.jsx": """import { Link } from "react-router-dom";
import { ShoppingCart, Heart } from "lucide-react";

export default function NavDrawer() {
  const cartCount = JSON.parse(localStorage.getItem("cart") || "[]").length;
  const wishCount = JSON.parse(localStorage.getItem("wishlist") || "[]").length;

  return (
    <div className="fixed top-0 left-0 w-full bg-white shadow-md flex justify-between px-4 py-2 z-50">
      <Link to="/home" className="font-bold text-xl text-maroon-700">DesiNext</Link>
      <div className="flex gap-4">
        <Link to="/wishlist">
          <Heart />
          <span>{wishCount}</span>
        </Link>
        <Link to="/cart">
          <ShoppingCart />
          <span>{cartCount}</span>
        </Link>
      </div>
    </div>
  );
}""",

    "src/components/ProductCard.jsx": """export default function ProductCard({ product }) {
  const addTo = (type) => {
    let items = JSON.parse(localStorage.getItem(type) || "[]");
    items.push(product);
    localStorage.setItem(type, JSON.stringify(items));
    alert(`Added to ${type}`);
  };

  return (
    <div className="border p-4 rounded-lg shadow">
      <h3 className="font-bold">{product.name}</h3>
      <p>₹{product.price}</p>
      <div className="flex gap-2 mt-2">
        <button onClick={() => addTo("cart")} className="bg-green-600 text-white px-2 py-1 rounded">
          Add to Cart
        </button>
        <button onClick={() => addTo("wishlist")} className="bg-pink-600 text-white px-2 py-1 rounded">
          Wishlist
        </button>
      </div>
    </div>
  );
}""",

    "src/components/VoiceAssistant.jsx": """import { useState } from "react";

export default function VoiceAssistant() {
  const [transcript, setTranscript] = useState("");

  const startListening = () => {
    const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
    recognition.lang = "en-IN";
    recognition.onresult = (event) => {
      const text = event.results[0][0].transcript;
      setTranscript(text);
      alert("AI Reply: Sure! Showing you results for " + text);
    };
    recognition.start();
  };

  return (
    <div className="mt-4">
      <button onClick={startListening} className="bg-maroon-700 text-white px-4 py-2 rounded-full">
        🎙️ Ask GramSaathi
      </button>
      {transcript && <p className="mt-2">You said: <strong>{transcript}</strong></p>}
    </div>
  );
}""",

    ".gitignore": "node_modules\nbuild\n.env",
    "package.json": """{
  "name": "desinext-app",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "framer-motion": "^11.0.0",
    "lucide-react": "^0.292.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "react-router-dom": "^6.10.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  }
}""",

    "tailwind.config.js": """module.exports = {
  content: ["./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {
      colors: {
        maroon: {
          700: '#800000'
        }
      }
    },
  },
  plugins: [],
};""",

    "postcss.config.js": """module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};""",

    "README.md": "# DesiNext App\n\nThis is the full-fledged DesiNext App built using React, Tailwind CSS, Framer Motion, and React Router."
}

# Create folders
for folder in folders:
    os.makedirs(os.path.join(base_dir, folder), exist_ok=True)

# Write files
for filepath, content in files.items():
    full_path = os.path.join(base_dir, filepath)
    os.makedirs(os.path.dirname(full_path), exist_ok=True)
    with open(full_path, "w") as f:
        f.write(content)

# Zip the directory
zip_path = f"{base_dir}.zip"
with zipfile.ZipFile(zip_path, 'w') as zipf:
    for root, dirs, files_in_dir in os.walk(base_dir):
        for file in files_in_dir:
            full_path = os.path.join(root, file)
            arcname = os.path.relpath(full_path, base_dir)
            zipf.write(full_path, arcname)

zip_path
