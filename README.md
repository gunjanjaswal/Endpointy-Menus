# EndPointy Menus

A WordPress plugin that exposes navigation menus via custom REST API endpoints. Perfect for headless WordPress, decoupled applications, and external integrations.

## 🚀 Features

- ✅ Expose all WordPress menus via REST API
- ✅ Get menus by ID, slug, or location (e.g., `primary`, `footer`)
- ✅ Nested menu hierarchy support with `?nested=true`
- ✅ Rich item meta with `?meta=true` (description, featured image, excerpt)
- ✅ Field filtering with `?fields=id,title,url` for lean payloads
- ✅ List all registered menu locations
- ✅ Built-in response caching (auto-flushed when menus change)
- ✅ Admin settings page (Settings → EndPointy Menus)
- ✅ Optional API-key authentication + CORS allow-list
- ✅ Developer filter hook (`endpointy_menus_item`)
- ✅ Clean, structured JSON output
- ✅ Lightweight and performant

## 📦 Installation

1. Download or clone this repository
2. Copy the `endpointy-menus` folder to your WordPress `wp-content/plugins` directory
3. Activate the plugin from **WP Admin → Plugins**
4. Ensure you have menus configured under **Appearance → Menus**

## 🔌 API Endpoints

Base namespace: `endpointy-menus/v1`

### Get All Menus
```
GET /wp-json/endpointy-menus/v1/menus
```
Returns all registered menus with their locations and items.

**Example Response:**
```json
[
  {
    "id": 2,
    "name": "Main Menu",
    "slug": "main-menu",
    "locations": ["primary"],
    "items": [...]
  }
]
```

### Get Single Menu by ID
```
GET /wp-json/endpointy-menus/v1/menus/{id}
```
Returns a specific menu and its items.

**Example:**
```
GET /wp-json/endpointy-menus/v1/menus/2
```

### Get Single Menu by Slug
```
GET /wp-json/endpointy-menus/v1/menus/slug/{slug}
```
Returns a specific menu and its items by slug.

**Example:**
```
GET /wp-json/endpointy-menus/v1/menus/slug/main-menu
```

### Get All Menu Locations
```
GET /wp-json/endpointy-menus/v1/locations
```
Returns all registered menu locations with assigned menus.

**Example Response:**
```json
[
  {
    "location": "primary",
    "description": "Primary Menu",
    "menu_id": 2,
    "menu_name": "Main Menu"
  }
]
```

### Get Menu by Location
```
GET /wp-json/endpointy-menus/v1/locations/{location}
```
Returns the menu assigned to a specific location.

**Example:**
```
GET /wp-json/endpointy-menus/v1/locations/primary
```

## 🌳 Nested Menu Structure

Add `?nested=true` to any menu endpoint to get a hierarchical tree structure:

```
GET /wp-json/endpointy-menus/v1/menus/2?nested=true
GET /wp-json/endpointy-menus/v1/locations/primary?nested=true
```

**Flat structure (default):**
```json
{
  "items": [
    {"id": 1, "title": "Home", "parent": 0},
    {"id": 2, "title": "About", "parent": 0},
    {"id": 3, "title": "Team", "parent": 2}
  ]
}
```

**Nested structure (`?nested=true`):**
```json
{
  "items": [
    {
      "id": 1,
      "title": "Home",
      "children": []
    },
    {
      "id": 2,
      "title": "About",
      "children": [
        {
          "id": 3,
          "title": "Team",
          "children": []
        }
      ]
    }
  ]
}
```

## ⚙️ Query Parameters

All menu endpoints accept these optional parameters (combinable):

| Param | Example | Description |
|-------|---------|-------------|
| `nested` | `?nested=true` | Hierarchical tree with `children` arrays |
| `meta` | `?meta=true` | Adds `description`, `attr_title`, `current`, `featured_image`, `excerpt` |
| `fields` | `?fields=id,title,url` | Return only the listed item fields (`children` always kept in nested mode) |

```
GET /wp-json/endpointy-menus/v1/menus/2?nested=true&meta=true&fields=id,title,url,featured_image,children
```

## 🛡️ Settings, Caching & Auth

A settings page lives at **Settings → EndPointy Menus**:

- **Response caching** — menu responses are cached as transients with a configurable lifetime and automatically flushed whenever a menu is edited or deleted.
- **API key** — turn on *Require API key* to protect the endpoints. Send the key as an `X-API-Key` header or an `api_key` query parameter:
  ```bash
  curl -H "X-API-Key: YOUR_KEY" https://your-site.com/wp-json/endpointy-menus/v1/menus
  ```
- **CORS** — add allowed origins (one per line, or `*`) to send `Access-Control-Allow-Origin` headers for browser clients.
- **Rate limiting** — cap requests per minute per IP (0 = off). Responses include `X-RateLimit-Limit` / `X-RateLimit-Remaining`; over-limit requests get `429` with `Retry-After`.
- **ETag / Last-Modified** — conditional-request support. Send `If-None-Match` or `If-Modified-Since` and unchanged menus return `304 Not Modified`:
  ```bash
  curl -H 'If-None-Match: "<etag>"' https://your-site.com/wp-json/endpointy-menus/v1/menus
  ```

## 🧩 Developer Hooks

```php
// Add or modify fields on every menu item.
add_filter( 'endpointy_menus_item', function ( $data, $item, $meta ) {
    $data['acf'] = get_fields( $item );
    return $data;
}, 10, 3 );
```

## 📝 Menu Item Properties

Each menu item includes:

| Property | Type | Description |
|----------|------|-------------|
| `id` | int | Menu item ID |
| `title` | string | Display title |
| `url` | string | Link URL |
| `parent` | int | Parent item ID (0 for top-level) |
| `order` | int | Menu order |
| `type` | string | Item type (post_type, taxonomy, custom, etc.) |
| `object` | string | Object type (page, post, category, etc.) |
| `object_id` | int | ID of the linked object |
| `target` | string | Link target (_blank, etc.) |
| `classes` | array | CSS classes |
| `xfn` | string | XFN relationship |
| `children` | array | Child items (only in nested mode) |
| `description` | string | Item description (only with `meta=true`) |
| `attr_title` | string | Title attribute (only with `meta=true`) |
| `current` | bool | Whether the item is the current page (only with `meta=true`) |
| `featured_image` | string\|null | Featured image URL for post-type items (only with `meta=true`) |
| `excerpt` | string | Post excerpt for post-type items (only with `meta=true`) |

## 💡 Usage Examples

### JavaScript (Fetch API)
```javascript
// Get primary menu
fetch('https://your-site.com/wp-json/endpointy-menus/v1/locations/primary')
  .then(response => response.json())
  .then(menu => console.log(menu));

// Get nested menu structure
fetch('https://your-site.com/wp-json/endpointy-menus/v1/menus/2?nested=true')
  .then(response => response.json())
  .then(menu => console.log(menu));
```

### React Example
```jsx
import { useEffect, useState } from 'react';

function Navigation() {
  const [menu, setMenu] = useState(null);

  useEffect(() => {
    fetch('https://your-site.com/wp-json/endpointy-menus/v1/locations/primary?nested=true')
      .then(res => res.json())
      .then(data => setMenu(data));
  }, []);

  if (!menu) return <div>Loading...</div>;

  return (
    <nav>
      {menu.items.map(item => (
        <a key={item.id} href={item.url}>{item.title}</a>
      ))}
    </nav>
  );
}
```

### cURL
```bash
# Get all menus
curl https://your-site.com/wp-json/endpointy-menus/v1/menus

# Get menu by location
curl https://your-site.com/wp-json/endpointy-menus/v1/locations/primary

# Get nested structure
curl "https://your-site.com/wp-json/endpointy-menus/v1/menus/2?nested=true"
```

## 🛠️ Development

### Requirements
- WordPress 5.0+
- PHP 7.0+

### File Structure
```
endpointy-menus/
├── endpointy-menus.php  # Main plugin file
├── readme.txt                # WordPress.org readme
└── README.md                 # GitHub readme
```

## 📄 License

GPL v2 or later

## 👨‍💻 Author

**Gunjan Jaswal**

- Website: [https://gunjanjaswal.me](https://gunjanjaswal.me)
- GitHub: [https://github.com/gunjanjaswal/Endpointy-Menus](https://github.com/gunjanjaswal/Endpointy-Menus)

## ☕ Support

If you find this plugin useful, consider supporting the developer:

[![Support on Ko-fi](https://img.shields.io/badge/Ko--fi-Support-FF5E5B?style=for-the-badge&logo=ko-fi&logoColor=white)](https://ko-fi.com/gunjanjaswal)

## 📜 Changelog

### 1.2.0
- Added admin settings page (Settings → EndPointy Menus).
- Added response caching with configurable lifetime and auto-flush on menu changes.
- Added optional API-key authentication and configurable CORS allow-list.
- Added per-IP rate limiting with `X-RateLimit-*` headers and `429` responses.
- Added ETag / Last-Modified conditional requests (`304 Not Modified`).
- Added `/menus/slug/<slug>` endpoint.
- Added `meta=true` parameter (description, featured image, excerpt, current flag).
- Added `fields=` parameter for partial responses.
- Added `endpointy_menus_item` developer filter.
- Added menu `count` and location `menu_slug` to responses.

### 1.1.1
- Updated "Tested up to" to WordPress 7.0.
- Updated donation link to Ko-fi (https://ko-fi.com/gunjanjaswal).

### 1.1.0
- Added support for filtering menus by location.
- Added nested menu hierarchy with `nested=true` query parameter.
- Added `/locations` endpoint to list all menu locations.
- Added `/locations/<location>` endpoint to get menu by location.

### 1.0.0
- Initial release.

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the [issues page](https://github.com/gunjanjaswal/Endpointy-Menus/issues).

## ⭐ Show Your Support

Give a ⭐️ if this project helped you!

---

Made with ❤️ by [Gunjan Jaswal](https://gunjanjaswal.me)
