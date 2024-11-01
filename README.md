
# Image Uploader

A simple React application for uploading images to an external API. This app allows users to select an image file and upload it, displaying a success or failure message.

## Table of Contents

- [Features](#features)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Contributing](#contributing)
- [License](#license)

## Features

- Upload image files via a simple user interface
- Display progress and success/failure messages to the user
- Easily configurable API URL and settings

## Getting Started

### Prerequisites

To run this application, ensure you have the following installed:

- [Node.js](https://nodejs.org/) (version 14 or higher)
- [npm](https://www.npmjs.com/) or [yarn](https://yarnpkg.com/)
- API endpoint: Create an API using Cloudflare Workers to handle image uploads. Example code for the worker is provided below.

### Installation

1. Clone the repository:

   ```bash
   git clone https://github.com/your-username/image-uploader.git
   cd image-uploader
   ```

2. Install dependencies:

   ```bash
   npm install
   # or if you prefer yarn
   yarn install
   ```

3. Start the development server:

   ```bash
   npm start
   # or if you prefer yarn
   yarn start
   ```

### Configuration

This application requires an API endpoint for uploading images. Update the `API_URL` constant in `ImageUploader.js` with your API endpoint.

```javascript
const API_URL = 'https://your-api-url.com/';
```

### Cloudflare Worker for Image API

Use the following Cloudflare Worker script to set up your image upload API:

```javascript
const API_URL = "https://api.cloudflare.com/client/v4/accounts/ACC_NUM/images/v1";
const TOKEN = "API_TOKEN";

export default {
  async fetch(request, env) {
    if (request.method !== 'POST') {
      return new Response("Only POST requests are allowed", { status: 405 });
    }

    // Parse form data from the incoming request
    const formData = await request.formData();
    const file = formData.get('file');

    if (!file) {
      return new Response("No file found in the request", { status: 400 });
    }

    // Create new FormData to send to the Cloudflare API
    const apiFormData = new FormData();
    apiFormData.append('file', file, file.name);

    // Send the image to the Cloudflare Images API
    const response = await fetch(API_URL, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${TOKEN}`,
      },
      body: apiFormData,
    });

    const result = await response.json();

    if (response.ok && result.success) {
      const variants = result.result.variants;
      console.log("Image Variants:", variants);
      return new Response(JSON.stringify({ message: "Image uploaded successfully!", variants }), {
        headers: {
          'Content-Type': 'application/json',
          'Access-Control-Allow-Origin': '*', // Adjust as needed
          'Access-Control-Allow-Methods': 'POST, OPTIONS',
          'Access-Control-Allow-Headers': 'Authorization, Content-Type',
        },
      });
    } else {
      console.error("Upload failed:", result.errors);
      return new Response(`Upload failed: ${result.errors[0]?.message || "Unknown error"}`, {
        status: 500,
        headers: { 'Content-Type': 'text/plain' },
      });
    }
  }
};
```

In this worker script:
- **API_URL**: Your Cloudflare Images API URL.
- **TOKEN**: Your Cloudflare API token.

