
# 🔗 FCRF Third-Party API Integration Guide

This guide describes how to integrate with the FCRF IDU Editor via API and iFrame.

---

## Integration Flow Diagram

![mermaid_20250513_476450](https://github.com/user-attachments/assets/586585aa-6bcc-4b90-9c1f-47471d0e0ee1)

## 🛡️ Step 1: Authenticate and Generate a Token

Use the following API to generate an authentication token with a 30-minute expiry.

**Endpoint:**  
`POST /api/login`

**Headers Required:**
```
X-fcrf-auth: <your-auth-header>
```

**Request Body:**
```json
{
  "user_id": "xxxxxxxxxxxxxxxxx"
}
```

**Response Example:**
```json
{
  "data": {
    "status": 200,
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

## ✏️ Step 2: Load the IDU Editor via iFrame

You can open the IDU Editor using an iFrame with the **diamond parameters** or **diamond parameters** with **requestId**  (for updating existing records).

### Option A: Load Editor with Diamond Parameters

**Base URL:**  
`https://d1kwyv57dsepak.cloudfront.net/idu-editor`

**Required Query Parameters:**

| Parameter     | Type    | Description                            | Example           |
|--------------|---------|----------------------------------------|-------------------|
| `token`      | string  | API token generated in Step 1          | `abc123`          |
| `origin_url`      | string  | Use window.location.origin          | `https://ld97yq.csb.app`          |
| `carat_weight` | number  | Diamond weight in carats              | `1.3`             |
| `shape`      | string  | Shape of the diamond                   | `Pear`            |
| `color`      | string  | Color grade                            | `Pink`            |
| `saturation` | string  | Saturation description                 | `Fancy Vivid`     |
| `clarity`    | string  | Clarity grade                          | `VS1`             |
| `fluorescence` | string | Fluorescence details                  | `Strong Yellow`   |
| `length`     | number  | Diamond length in mm                   | `15.21`           |
| `width`      | number  | Diamond width in mm                    | `11.01`           |
| `polish`     | string  | Polish grade                           | `Excellent`       |
| `symmetry`   | string  | Symmetry grade                         | `Very Good`       |
| `trueFaceUp` | string  | Face-up orientation setting            | `Standard`        |
| `singlePair` | string  | Single or pair selection               | `Single`          |

### Option B: Load Editor with Request ID and Diamond Details

You can pass a `requestId` and `diamond details` to update an existing IDU record.

| Parameter     | Type    | Description                            |
|---------------|---------|----------------------------------------|
| `requestId`   | string  | Optional. ID of an existing request    |

> ⚠️ If using `requestId`, it's recommended to pass it **before** other parameters in the query string.

---

## 🌐 Example URLs

### Without `requestId`
```
https://d1kwyv57dsepak.cloudfront.net/idu-editor?token=abc123&origin_url=https://ld97yq.csb.app&carat_weight=1.3&shape=Pear&color=Pink&saturation=Fancy%20Vivid&clarity=VS1&fluorescence=Strong%20Yellow&length=15.21&width=11.01&polish=Excellent&symmetry=Very%20Good&trueFaceUp=Standard&singlePair=Single
```

### With `requestId`
```
https://d1kwyv57dsepak.cloudfront.net/idu-editor?requestId=866564668678786&token=abc123&origin_url=https://ld97yq.csb.app&carat_weight=1.3&shape=Pear&color=Pink&saturation=Fancy%20Vivid&clarity=VS1&fluorescence=Strong%20Yellow&length=15.21&width=11.01&polish=Excellent&symmetry=Very%20Good&trueFaceUp=Standard&singlePair=Single
```

---

## 🧩 Step 3: Embed the Editor iFrame

Use the URL in an `<iframe>` to load the editor in your page:

```html
<iframe 
  src="https://d1kwyv57dsepak.cloudfront.net/idu-editor?token=abc123&carat_weight=1.3&shape=Pear&color=Pink&saturation=Fancy%20Vivid&clarity=VS1&fluorescence=Strong%20Yellow&length=15.21&width=11.01&polish=Excellent&symmetry=Very%20Good&trueFaceUp=Standard&singlePair=Single"
  width="100%"
  height="800"
  frameborder="0"
></iframe>
```

After embedding, a popup will open with IDU options.

---

## 💰 Step 4: Generate Price via IDU Editor

When the user completes editing and selects **"NEW IDU"**, they can generate a new price.

Two action buttons will be available:

- **Adjust IDU** – Allows users to change parameters and generate a new price.
- **Apply Price** – Sends the final IDU data to the parent website.

---

## 🔄 Step 5: Capture Data from the iFrame

Use `window.postMessage` to securely receive data back from the iFrame:

```javascript
useEffect(() => {
  const handleMessage = (event) => {
    if (event.origin !== "https://d1kwyv57dsepak.cloudfront.net") return;

    console.log("Received data from iframe:", event.data);
    setReceivedData(event.data);
  };

  window.addEventListener("message", handleMessage);

  return () => {
    window.removeEventListener("message", handleMessage);
  };
}, []);
```

## 📦 Example Received Data Format:


```javascript
{
  "update_following_IDU": true,
  "requestId": "866564668678786",
  "price_low_range": 5500,
  "price_high_range": 7000,
  "colorInnerGrade": 3,
  "colorDispersion": 2,
  "colorUndertone": 4
}

```

---



## 📌 Notes

- Always validate the `event.origin` when receiving messages.
- The token is valid for 30 minutes. Regenerate it if it expires.
- `requestId` is optional but useful for updating or tracking IDU edits.
