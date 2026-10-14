# Axios

Axios is an HTTP client library. You give it a URL and options, and it sends the request and returns a Promise with the response. React Native already has the built-in `fetch`, but Axios gives you a more convenient API.

## Basic usage

```js
import axios from "axios";

// GET
const res = await axios.get("https://api.example.com/plants");
console.log(res.data);   // already-parsed JSON
console.log(res.status); // 200

// GET with query params -> /plants?page=2&limit=10
await axios.get("https://api.example.com/plants", {
  params: { page: 2, limit: 10 },
});

// POST with a JSON body (Axios stringifies it and sets Content-Type for you)
await axios.post("https://api.example.com/plants", {
  name: "Aloe",
  waterEveryDays: 7,
});

// Custom headers
await axios.get("https://api.example.com/me", {
  headers: { Authorization: "Bearer my-token" },
});
```

## Create one configured instance and reuse it

```js
// src/api/client.js
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 10000,
  headers: { "Content-Type": "application/json" },
});
```

```js
// src/api/plants.js
import { api } from "./client";

export const getPlants = async () => {
  const { data } = await api.get("/plants");
  return data;
};

export const createPlant = async (plant) => {
  const { data } = await api.post("/plants", plant);
  return data;
};

export const deletePlant = (id) => api.delete(`/plants/${id}`);
```

## How Axios differs from fetch

```js
// fetch: two steps, manual JSON parsing, no error on 404/500
const res = await fetch(url);
const data = await res.json();

// axios: data is already parsed, and non-2xx responses THROW
const { data } = await axios.get(url);
```

| Aspect | `fetch` | `axios` |
| --- | --- | --- |
| Steps to get data | Two: request, then `res.json()` | One: `data` is already parsed |
| JSON parsing | Manual (`res.json()`) | Automatic |
| 404 / 500 responses | Do not throw; you check `res.ok` yourself | Non-2xx responses throw |
| Request body (JSON) | You stringify it and set headers yourself | Stringified and `Content-Type` set for you |
