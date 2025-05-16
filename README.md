# 🔗 FCRF Third-Party API Integration Guide

This guide explains how to integrate the FCRF IDU Editor using API authentication and an embedded iFrame.

---

## 🧭 Integration Flow Overview

![Integration Flow Diagram](https://github.com/user-attachments/assets/586585aa-6bcc-4b90-9c1f-47471d0e0ee1)

---

## 🛡️ Step 1: Authenticate and Generate Token

Generate an authentication token (valid for 30 minutes) using the following endpoint.

### **Endpoint**
`POST /api/login`

### **Headers**
```http
X-fcrf-auth: <your-auth-header>
```

### **Request Body**
```json
{
  "user_id": "xxxxxxxxxxxxxxxxx"
}
```

### **Sample Response**
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

### **Base iFrame URL**
`https://d1kwyv57dsepak.cloudfront.net/idu-editor`

### **Common Query Parameters**

| Parameter      | Type   | Description                  | Example                    |
|----------------|--------|------------------------------|----------------------------|
| `token`        | string | Token generated in Step 1    | `abc123`                   |
| `origin_url`   | string | Current window origin        | `https://ld97yq.csb.app`   |

---

### ✅ Option A: Initial Load with Diamond Parameters

Load the editor with initial diamond details. No pre-selected IDU options will be used.

#### **Required Parameters**

| Parameter         | Type    | Description                      | Example          |
|------------------|---------|----------------------------------|------------------|
| carat_weight     | number  | Diamond weight in carats         | 1.3              |
| shape            | string  | Shape of the diamond             | Pear             |
| color            | string  | Color grade                      | Pink             |
| saturation       | string  | Saturation level                 | Fancy Vivid      |
| clarity          | string  | Clarity grade                    | VS1              |
| fluorescence     | string  | Fluorescence description         | Strong Yellow    |
| length           | number  | Length in mm                     | 15.21            |
| width            | number  | Width in mm                      | 11.01            |
| polish           | string  | Polish grade                     | Excellent        |
| symmetry         | string  | Symmetry grade                   | Very Good        |
| trueFaceUp       | string  | Face-up orientation              | Standard         |
| singlePair       | string  | Single or Pair                   | Single           |

> 📝 **Note:** On the "Color Undertone" screen (3rd step), if the checkbox is selected, the chosen IDU option is saved for the next session. To use this saved selection in future requests, add `update_following_idu=true` to the query parameters.

#### **Example URL**
```
https://d1kwyv57dsepak.cloudfront.net/idu-editor?token=abc123&origin_url=https://ld97yq.csb.app&carat_weight=1.3&shape=Pear&color=Pink&saturation=Fancy%20Vivid&clarity=VS1&fluorescence=Strong%20Yellow&length=15.21&width=11.01&polish=Excellent&symmetry=Very%20Good&trueFaceUp=Standard&singlePair=Single
```

---

### 🔄 Option B: Next Calculation with `update_following_idu=true`

Use this if you've saved IDU options from a previous session. The editor will skip popups and return a calculated price.

#### **Additional Parameter**
| Parameter            | Type    | Description                          | Example |
|----------------------|---------|--------------------------------------|---------|
| update_following_idu | boolean | Use stored IDU options if available  | true    |

#### **Example URL**
```
https://d1kwyv57dsepak.cloudfront.net/idu-editor?token=abc123&origin_url=https://ld97yq.csb.app&update_following_idu=true&carat_weight=1.3&shape=Pear&color=Pink&saturation=Fancy%20Vivid&clarity=VS1&fluorescence=Strong%20Yellow&length=15.21&width=11.01&polish=Excellent&symmetry=Very%20Good&trueFaceUp=Standard&singlePair=Single
```

---

### 🧾 Option C: Load Editor with `requestId` for Editing

Pass a valid `requestId` to load a previously submitted request.

#### **Parameters**

| Parameter            | Type    | Description                                         | Example     |
|----------------------|---------|-----------------------------------------------------|-------------|
| requestId            | number  | Existing request ID                                 | 786378773767 |
| update_following_idu | boolean | Optional - open with checkbox checked or not        | true/false  |

#### **Example URL**
```
https://d1kwyv57dsepak.cloudfront.net/idu-editor?token=abc123&origin_url=https://ld97yq.csb.app&requestId=786378773767&carat_weight=1.3&shape=Pear&color=Pink&saturation=Fancy%20Vivid&clarity=VS1&fluorescence=Strong%20Yellow&length=15.21&width=11.01&polish=Excellent&symmetry=Very%20Good&trueFaceUp=Standard&singlePair=Single
```

---

## 🧩 Step 3: Embed the Editor via iFrame

### Example HTML iFrames

#### Initial Load
```html
<iframe 
  src="https://d1kwyv57dsepak.cloudfront.net/idu-editor?token=abc123&origin_url=https://ld97yq.csb.app&carat_weight=1.3&shape=Pear&color=Pink..."
  width="100%" height="800" frameborder="0">
</iframe>
```

#### With `update_following_idu`
```html
<iframe 
  src="https://d1kwyv57dsepak.cloudfront.net/idu-editor?token=abc123&origin_url=https://ld97yq.csb.app&update_following_idu=true&carat_weight=1.3..."
  width="100%" height="800" frameborder="0">
</iframe>
```

#### With `requestId`
```html
<iframe 
  src="https://d1kwyv57dsepak.cloudfront.net/idu-editor?token=abc123&origin_url=https://ld97yq.csb.app&requestId=787887878789&carat_weight=1.3..."
  width="100%" height="800" frameborder="0">
</iframe>
```

> A popup will open after the iFrame loads for IDU selection.

---

## 💰 Step 4: User Interaction in the Editor

Once editing is complete, users will see:

- **Adjust IDU**: Modify IDU values and recalculate.
- **Apply Price**: Send the finalized data back to your site.
- **Close Popup**: Exit without applying changes.

---

## 🔄 Step 5: Receive Data from the iFrame

Use `postMessage` to listen for results from the editor.

### Example Code (React)
```javascript
useEffect(() => {
  const handleMessage = (event) => {
    if (event.origin !== "https://d1kwyv57dsepak.cloudfront.net") return;
    console.log("Received data from iframe:", event.data);
    setReceivedData(event.data);
  };

  window.addEventListener("message", handleMessage);
  return () => window.removeEventListener("message", handleMessage);
}, []);
```

### Sample Received Data
```json
{
  "update_following_IDU": true,
  "requestId": "866564668678786",
  "price_low_range": 5500,
  "price_high_range": 7000
}
```

---

## 📌 Notes

- ✅ Always validate the origin using `event.origin`.
- ⏳ Token expires in 30 minutes; regenerate as needed.
- 🧾 `requestId` is optional but useful for edits and tracking.
- 💡 If `update_following_idu=true` is passed and no session data exists, the system will open the popup as a fallback.
