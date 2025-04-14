# E-commerce-system.py
// React Frontend integrated with Backend API (Python Example: e_commerce_system.py)

import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import axios from "axios";

const API_BASE = "http://localhost:5000"; // Flask backend URL

export default function App() {
  const [products, setProducts] = useState([]);
  const [cart, setCart] = useState([]);
  const [search, setSearch] = useState("");
  const [loggedIn, setLoggedIn] = useState(false);
  const [username, setUsername] = useState("");
  const [email, setEmail] = useState("");

  useEffect(() => {
    axios.get(`${API_BASE}/products`).then((res) => setProducts(res.data));
  }, []);

  const addToCart = (product) => {
    const exists = cart.find((item) => item.id === product.id);
    if (exists) {
      setCart(
        cart.map((item) =>
          item.id === product.id ? { ...item, quantity: item.quantity + 1 } : item
        )
      );
    } else {
      setCart([...cart, { ...product, quantity: 1 }]);
    }
  };

  const removeFromCart = (id) => {
    setCart(cart.filter((item) => item.id !== id));
  };

  const total = cart.reduce((sum, item) => sum + item.price * item.quantity, 0);

  const filteredProducts = products.filter((product) =>
    product.name.toLowerCase().includes(search.toLowerCase())
  );

  const handleLogin = () => {
    if (username.trim() && email.trim()) {
      axios.post(`${API_BASE}/login`, { name: username, email }).then(() => {
        setLoggedIn(true);
      });
    }
  };

  const handleCheckout = () => {
    axios
      .post(`${API_BASE}/order`, {
        customer: { name: username, email },
        cart,
      })
      .then((res) => {
        alert(res.data.message);
        setCart([]);
      });
  };

  return (
    <div className="p-6">
      {!loggedIn ? (
        <div className="max-w-md mx-auto mt-20">
          <h1 className="text-2xl font-bold mb-4">Login</h1>
          <input
            type="text"
            placeholder="Username"
            className="border p-2 w-full mb-2"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
          />
          <input
            type="email"
            placeholder="Email"
            className="border p-2 w-full mb-4"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
          />
          <Button onClick={handleLogin}>Login</Button>
        </div>
      ) : (
        <>
          <h1 className="text-3xl font-bold mb-4">Welcome, {username}</h1>

          <input
            type="text"
            placeholder="Search products..."
            className="border p-2 mb-4 w-full"
            value={search}
            onChange={(e) => setSearch(e.target.value)}
          />

          <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
            {filteredProducts.map((product) => (
              <Card key={product.id}>
                <CardContent className="p-4">
                  <img src={product.image || "https://via.placeholder.com/100"} alt={product.name} className="mb-2 rounded h-32 w-full object-contain" />
                  <h2 className="text-xl font-semibold">{product.name}</h2>
                  <p className="text-gray-600">${product.price}</p>
                  <Button className="mt-2" onClick={() => addToCart(product)}>
                    Add to Cart
                  </Button>
                </CardContent>
              </Card>
            ))}
          </div>

          <div className="mt-10">
            <h2 className="text-2xl font-bold mb-2">Shopping Cart</h2>
            {cart.length === 0 ? (
              <p>Your cart is empty.</p>
            ) : (
              <div className="space-y-2">
                {cart.map((item) => (
                  <div key={item.id} className="flex justify-between items-center">
                    <span>
                      {item.name} x {item.quantity}
                    </span>
                    <span>${(item.price * item.quantity).toFixed(2)}</span>
                    <Button variant="destructive" onClick={() => removeFromCart(item.id)}>
                      Remove
                    </Button>
                  </div>
                ))}
                <div className="font-semibold mt-2">Total: ${total.toFixed(2)}</div>
                <Button className="mt-2" onClick={handleCheckout}>Checkout</Button>
              </div>
            )}
          </div>
        </>
      )}
    </div>
  );
}
