# Rental Property Management Dashboard - Complete Project Export

## Project Structure
Create these folders and files in your local project:

```
rental-property-dashboard/
├── package.json
├── dashboard.html
├── index.html
├── css/
│   └── styles.css
├── js/
│   ├── api.js
│   ├── app.js
│   ├── properties.js
│   └── supabase.js
├── server/
│   ├── db-api.js
│   ├── storage.js
│   ├── notifications.js
│   ├── scheduler.js
│   ├── api.js
│   └── simple-api.js
└── shared/
    └── schema.js
```

## Setup Instructions
1. Create a new folder on your computer
2. Copy each file content below into the corresponding file
3. Run `npm install` to install dependencies
4. Set up your PostgreSQL database and environment variables
5. Run `node server/db-api.js` to start the server

## Environment Variables Required
```
DATABASE_URL=your_postgresql_connection_string
RESEND_API_KEY=your_resend_api_key
SIGNWELL_API_KEY=your_signwell_api_key
```

---

## FILE CONTENTS START HERE

### package.json
```json
{
  "name": "rental-property-dashboard",
  "version": "1.0.0",
  "main": "server/db-api.js",
  "scripts": {
    "start": "node server/db-api.js",
    "dev": "node server/db-api.js"
  },
  "keywords": ["rental", "property", "management", "dashboard"],
  "author": "",
  "license": "ISC",
  "description": "Rental Property Management Dashboard",
  "dependencies": {
    "@sendgrid/mail": "^8.1.5",
    "@supabase/supabase-js": "^2.50.0",
    "@types/pg": "^8.15.4",
    "csv-writer": "^1.6.0",
    "drizzle-orm": "^0.44.2",
    "pdfkit": "^0.17.1",
    "pg": "^8.16.2",
    "postgres": "^3.4.7",
    "resend": "^4.6.0"
  }
}
```

### shared/schema.js
```javascript
import { pgTable, serial, text, numeric, timestamp, integer, json } from 'drizzle-orm/pg-core';

export const properties = pgTable('properties', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  address: text('address').notNull(),
  monthly_rent: numeric('monthly_rent', { precision: 10, scale: 2 }).notNull(),
  bedrooms: integer('bedrooms').default(1),
  bathrooms: integer('bathrooms').default(1),
  sqft: integer('sqft'),
  photos: json('photos').default([]),
  created_at: timestamp('created_at').defaultNow().notNull(),
});
```

### js/api.js
```javascript
/**
 * API configuration and utilities
 */

// API base URL
const API_BASE_URL = window.location.origin + '/api';

/**
 * Make HTTP request to API
 */
async function apiRequest(endpoint, options = {}) {
    const url = `${API_BASE_URL}${endpoint}`;
    const config = {
        headers: {
            'Content-Type': 'application/json',
            ...options.headers
        },
        ...options
    };

    if (config.body && typeof config.body === 'object') {
        config.body = JSON.stringify(config.body);
    }

    const response = await fetch(url, config);
    
    if (!response.ok) {
        const errorData = await response.json().catch(() => ({ error: 'Request failed' }));
        throw new Error(errorData.error || `HTTP ${response.status}`);
    }

    return response.json();
}

/**
 * Database operations for properties using API
 */
const PropertyService = {
    /**
     * Fetch all properties from the database
     * @returns {Promise<Array>} Array of property objects
     */
    async getAll() {
        try {
            return await apiRequest('/properties');
        } catch (error) {
            console.error('Error fetching properties:', error);
            throw new Error(`Failed to fetch properties: ${error.message}`);
        }
    },

    /**
     * Create a new property
     * @param {Object} property - Property data to insert
     * @returns {Promise<Object>} Created property object
     */
    async create(property) {
        try {
            // Validate required fields
            if (!property.name || !property.address || !property.monthly_rent) {
                throw new Error('Name, address, and monthly rent are required');
            }

            // Ensure monthly_rent is a number
            const propertyData = {
                ...property,
                monthly_rent: parseFloat(property.monthly_rent)
            };

            return await apiRequest('/properties', {
                method: 'POST',
                body: propertyData
            });
        } catch (error) {
            console.error('Error creating property:', error);
            throw new Error(`Failed to create property: ${error.message}`);
        }
    },

    /**
     * Update an existing property
     * @param {number} id - Property ID to update
     * @param {Object} updates - Property data to update
     * @returns {Promise<Object>} Updated property object
     */
    async update(id, updates) {
        try {
            // Validate required fields
            if (!updates.name || !updates.address || !updates.monthly_rent) {
                throw new Error('Name, address, and monthly rent are required');
            }

            // Ensure monthly_rent is a number
            const propertyData = {
                ...updates,
                monthly_rent: parseFloat(updates.monthly_rent)
            };

            return await apiRequest(`/properties/${id}`, {
                method: 'PUT',
                body: propertyData
            });
        } catch (error) {
            console.error('Error updating property:', error);
            throw new Error(`Failed to update property: ${error.message}`);
        }
    },

    /**
     * Delete a property
     * @param {number} id - Property ID to delete
     * @returns {Promise<boolean>} Success status
     */
    async delete(id) {
        try {
            await apiRequest(`/properties/${id}`, {
                method: 'DELETE'
            });
            return true;
        } catch (error) {
            console.error('Error deleting property:', error);
            throw new Error(`Failed to delete property: ${error.message}`);
        }
    }
};

/**
 * Check API connection and health
 */
async function checkDatabaseConnection() {
    try {
        await apiRequest('/health');
        console.log('API connection successful');
        return true;
    } catch (error) {
        console.error('API connection failed:', error);
        throw new Error('Unable to connect to the API server');
    }
}

// Export for use in other modules
window.PropertyService = PropertyService;
window.checkDatabaseConnection = checkDatabaseConnection;
```

### css/styles.css
```css
/* Custom styles for the rental property management dashboard */

/* Smooth transitions for all interactive elements */
* {
    transition: all 0.2s ease-in-out;
}

/* Custom scrollbar styling */
::-webkit-scrollbar {
    width: 6px;
}

::-webkit-scrollbar-track {
    background: #f1f1f1;
}

::-webkit-scrollbar-thumb {
    background: #8b5cf6;
    border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
    background: #7c3aed;
}

/* Property card hover effects */
.property-card {
    transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
}

.property-card:hover {
    transform: translateY(-2px);
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
}

/* Button hover animations */
.btn-hover:hover {
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(139, 92, 246, 0.3);
}

/* Modal animations */
.modal-enter {
    animation: modalEnter 0.3s ease-out;
}

@keyframes modalEnter {
    from {
        opacity: 0;
        transform: scale(0.9) translateY(-10px);
    }
    to {
        opacity: 1;
        transform: scale(1) translateY(0);
    }
}

/* Toast notification animations */
.toast-enter {
    animation: toastEnter 0.3s ease-out;
}

.toast-exit {
    animation: toastExit 0.3s ease-in;
}

@keyframes toastEnter {
    from {
        opacity: 0;
        transform: translateX(100%);
    }
    to {
        opacity: 1;
        transform: translateX(0);
    }
}

@keyframes toastExit {
    from {
        opacity: 1;
        transform: translateX(0);
    }
    to {
        opacity: 0;
        transform: translateX(100%);
    }
}

/* Loading animation enhancements */
.loading-pulse {
    animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
    0%, 100% {
        opacity: 1;
    }
    50% {
        opacity: 0.5;
    }
}

/* Focus styles for accessibility */
input:focus,
textarea:focus,
button:focus {
    outline: none;
    ring: 2px;
    ring-color: #8b5cf6;
    ring-opacity: 0.5;
}

/* Form validation styles */
.error-input {
    border-color: #ef4444;
    box-shadow: 0 0 0 1px #ef4444;
}

.error-message {
    color: #ef4444;
    font-size: 0.875rem;
    margin-top: 0.25rem;
}

/* Custom card shadows */
.card-shadow {
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1), 0 1px 2px rgba(0, 0, 0, 0.06);
}

.card-shadow-lg {
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
}

/* Responsive design improvements */
@media (max-width: 640px) {
    .property-card {
        margin-bottom: 1rem;
    }
    
    .modal-content {
        margin: 1rem;
        max-height: 90vh;
        overflow-y: auto;
    }
}

/* Print styles */
@media print {
    .no-print {
        display: none !important;
    }
    
    .property-card {
        break-inside: avoid;
        margin-bottom: 1rem;
    }
}
```
