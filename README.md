import React, { useReducer, useContext, useEffect, useState } from "react";
import ReactDOM from "react-dom/client";
import {
  BrowserRouter as Router,
  Routes,
  Route,
  Link,
  useParams,
  useNavigate,
} from "react-router-dom";

/* ------------------ MOCK PRODUCTS ------------------ */
const products = [
  {
    id: 1,
    title: "Apple iPhone 15",
    price: 79999,
    rating: 4.8,
    image: "https://m.media-amazon.com/images/I/71d7rfSl0wL._SX679_.jpg",
    description: "Latest iPhone with A16 chip and stunning camera.",
  },
  {
    id: 2,
    title: "Samsung Galaxy S23",
    price: 74999,
    rating: 4.6,
    image: "https://m.media-amazon.com/images/I/71OXmy3NMCL._SX679_.jpg",
    description: "Premium Android smartphone with AMOLED display.",
  },
  {
    id: 3,
    title: "Sony Headphones",
    price: 19999,
    rating: 4.5,
    image: "https://m.media-amazon.com/images/I/61bK6PMOC3L._SX679_.jpg",
    description: "Noise cancelling wireless headphones.",
  },
];

/* ------------------ GLOBAL STATE ------------------ */
const initialState = {
  cart: [],
  user: JSON.parse(localStorage.getItem("user")) || null,
};

function reducer(state, action) {
  switch (action.type) {
    case "ADD_TO_CART":
      const existing = state.cart.find(i => i.id === action.payload.id);
      if (existing) {
        return {
          ...state,
          cart: state.cart.map(i =>
            i.id === action.payload.id
              ? { ...i, qty: i.qty + 1 }
              : i
          ),
        };
      }
      return { ...state, cart: [...state.cart, { ...action.payload, qty: 1 }] };

    case "REMOVE":
      return { ...state, cart: state.cart.filter(i => i.id !== action.payload) };

    case "QTY":
      return {
        ...state,
        cart: state.cart.map(i =>
          i.id === action.payload.id ? { ...i, qty: action.payload.qty } : i
        ),
      };

    case "LOGIN":
      localStorage.setItem("user", JSON.stringify(action.payload));
      return { ...state, user: action.payload };

    case "LOGOUT":
      localStorage.removeItem("user");
      return { ...state, user: null };

    default:
      return state;
  }
}

const AppContext = React.createContext();
const useApp = () => useContext(AppContext);

/* ------------------ NAVBAR ------------------ */
function Navbar() {
  const { state, dispatch } = useApp();
  return (
    <nav style={styles.nav}>
      <Link to="/" style={styles.logo}>Amazon</Link>
      <Link to="/cart">Cart ({state.cart.length})</Link>
      {state.user ? (
        <>
          <span>Hello, {state.user.name}</span>
          <button onClick={() => dispatch({ type: "LOGOUT" })}>Logout</button>
        </>
      ) : (
        <Link to="/login">Login</Link>
      )}
    </nav>
  );
}

/* ------------------ HOME ------------------ */
function Home() {
  const [search, setSearch] = useState("");
  const filtered = products.filter(p =>
    p.title.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div style={styles.container}>
      <input
        placeholder="Search products..."
        style={styles.search}
        onChange={e => setSearch(e.target.value)}
      />
      <div style={styles.grid}>
        {filtered.map(p => (
          <div key={p.id} style={styles.card}>
            <img src={p.image} alt="" style={styles.img} />
            <h4>{p.title}</h4>
            <p>₹{p.price}</p>
            <p>⭐ {p.rating}</p>
            <Link to={`/product/${p.id}`}>View</Link>
          </div>
        ))}
      </div>
    </div>
  );
}

/* ------------------ PRODUCT DETAIL ------------------ */
function ProductDetail() {
  const { id } = useParams();
  const { dispatch } = useApp();
  const product = products.find(p => p.id === Number(id));

  return (
    <div style={styles.container}>
      <img src={product.image} style={styles.bigImg} />
      <h2>{product.title}</h2>
      <p>{product.description}</p>
      <h3>₹{product.price}</h3>
      <button onClick={() => dispatch({ type: "ADD_TO_CART", payload: product })}>
        Add to Cart
      </button>
    </div>
  );
}

/* ------------------ CART ------------------ */
function Cart() {
  const { state, dispatch } = useApp();
  const total = state.cart.reduce((a, c) => a + c.price * c.qty, 0);

  return (
    <div style={styles.container}>
      <h2>Cart</h2>
      {state.cart.map(i => (
        <div key={i.id} style={styles.row}>
          <h4>{i.title}</h4>
          <input
            type="number"
            value={i.qty}
            min="1"
            onChange={e =>
              dispatch({ type: "QTY", payload: { id: i.id, qty: Number(e.target.value) } })
            }
          />
          <button onClick={() => dispatch({ type: "REMOVE", payload: i.id })}>
            Remove
          </button>
        </div>
      ))}
      <h3>Total: ₹{total}</h3>
      <Link to="/checkout">Proceed to Checkout</Link>
    </div>
  );
}

/* ------------------ CHECKOUT ------------------ */
function Checkout() {
  return (
    <div style={styles.container}>
      <h2>Checkout</h2>
      <input placeholder="Address" style={styles.input} />
      <input placeholder="City" style={styles.input} />
      <input placeholder="Pincode" style={styles.input} />
      <button>Place Order</button>
    </div>
  );
}

/* ------------------ LOGIN ------------------ */
function Login() {
  const [name, setName] = useState("");
  const { dispatch } = useApp();
  const nav = useNavigate();

  const login = () => {
    dispatch({ type: "LOGIN", payload: { name } });
    nav("/");
  };

  return (
    <div style={styles.container}>
      <h2>Login</h2>
      <input onChange={e => setName(e.target.value)} style={styles.input} />
      <button onClick={login}>Login</button>
    </div>
  );
}

/* ------------------ APP ------------------ */
function App() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <AppContext.Provider value={{ state, dispatch }}>
      <Router>
        <Navbar />
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/product/:id" element={<ProductDetail />} />
          <Route path="/cart" element={<Cart />} />
          <Route path="/checkout" element={<Checkout />} />
          <Route path="/login" element={<Login />} />
        </Routes>
      </Router>
    </AppContext.Provider>
  );
}

/* ------------------ STYLES ------------------ */
const styles = {
  nav: { display: "flex", gap: 20, padding: 15, background: "#131921", color: "#fff" },
  logo: { color: "orange", fontWeight: "bold", textDecoration: "none" },
  container: { padding: 20 },
  grid: { display: "grid", gridTemplateColumns: "repeat(auto-fit,minmax(200px,1fr))", gap: 20 },
  card: { border: "1px solid #ccc", padding: 10 },
  img: { width: "100%", height: 150, objectFit: "contain" },
  bigImg: { width: 300 },
  search: { padding: 10, width: "100%", marginBottom: 20 },
  row: { display: "flex", gap: 10, alignItems: "center" },
  input: { padding: 10, display: "block", marginBottom: 10 },
};

/* ------------------ RENDER ------------------ */
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);
