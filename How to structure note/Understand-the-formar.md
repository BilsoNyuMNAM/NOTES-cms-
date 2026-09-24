---
title: "Handling CORS with Cookies & Credentials in Express"
date: 2026-09-24
tags:
  - backend
  - express
  - security
  - cors
summary: "Why fetch() requests with cookies fail with CORS errors and how to configure cors origin + credentials properly."
---

# Handling CORS with Cookies & Credentials in Express

## 1. The Problem
When the frontend (`http://localhost:5173`) sends an authenticated API request with credentials (`include: 'credentials'`) to the backend (`http://localhost:3000`), the browser blocks the response with:

> `Access to fetch at '...' has been blocked by CORS policy: The value of the 'Access-Control-Allow-Origin' header in the response must not be the wildcard '*' when the request's credentials mode is 'include'.`

## 2. The Root Cause
By default, standard CORS setups often use wildcard origins (`*`). But browsers forbid wildcards whenever cookies or authorization headers are transmitted for security reasons.

## 3. The Solution

### Backend Configuration (`Express`)
Specify the exact frontend origin and enable `credentials: true`:

```ts
import cors from 'cors';
import express from 'express';

const app = express();

app.use(
  cors({
    origin: 'http://localhost:5173', // Exact origin, NEVER '*' with credentials
    credentials: true,               // Allows browser cookies to pass
  })
);