# Expres-Validator
A comprehensive guide to learning express-validator
# Comprehensive Express-Validator Guide

## Table of Contents
- [Introduction](#introduction)
- [Installation & Setup](#installation--setup)
- [Basic Concepts](#basic-concepts)
- [Getting Started](#getting-started)
- [Validation Methods](#validation-methods)
- [Built-in Validators](#built-in-validators)
- [Custom Validators](#custom-validators)
- [Sanitization](#sanitization)
- [Error Handling](#error-handling)
- [Schema Validation](#schema-validation)
- [Conditional Validation](#conditional-validation)
- [File Validation](#file-validation)
- [Advanced Techniques](#advanced-techniques)
- [Testing Validation](#testing-validation)
- [Best Practices](#best-practices)
- [Common Patterns](#common-patterns)
- [Performance Considerations](#performance-considerations)
- [Integration Examples](#integration-examples)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

## Introduction

Express-validator is a set of express.js middlewares that wraps validator.js validator and sanitizer functions. It provides a comprehensive solution for validating and sanitizing user input in Express applications, helping prevent security vulnerabilities and ensure data integrity.

### Why Express-Validator?
- **Security**: Prevents injection attacks and malicious input
- **Data Integrity**: Ensures data meets your application requirements
- **User Experience**: Provides clear error messages for form validation
- **Performance**: Lightweight and fast validation
- **Flexibility**: Highly customizable with extensive built-in validators
- **Integration**: Seamlessly integrates with Express middleware system

### Key Features
- Extensive built-in validation rules
- Custom validation functions
- Data sanitization capabilities
- Flexible error handling
- Schema-based validation
- Conditional validation
- File validation support

## Installation & Setup

### Installation
```bash
npm install express-validator
```

### Basic Setup
```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');

const app = express();

// Middleware to parse JSON
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

### Project Structure
```
project/
├── validators/
│   ├── userValidators.js
│   ├── productValidators.js
│   └── commonValidators.js
├── middleware/
│   └── validation.js
├── utils/
│   └── validationHelpers.js
└── app.js
```

## Basic Concepts

### Core Components
1. **Validation Chains**: Functions that define validation rules
2. **Validation Result**: Object containing validation errors
3. **Sanitizers**: Functions that clean and transform input data
4. **Custom Validators**: User-defined validation logic

### Validation Chain Methods
- `body()` - Validates request body fields
- `param()` - Validates URL parameters
- `query()` - Validates query string parameters
- `header()` - Validates request headers
- `cookie()` - Validates cookies

## Getting Started

### Simple Validation Example
```javascript
const { body, validationResult } = require('express-validator');

// Validation rules
const userValidation = [
  body('email')
    .isEmail()
    .withMessage('Please provide a valid email')
    .normalizeEmail(),
  
  body('password')
    .isLength({ min: 6 })
    .withMessage('Password must be at least 6 characters long')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .withMessage('Password must contain at least one uppercase letter, one lowercase letter, and one number')
];

// Route with validation
app.post('/register', userValidation, (req, res) => {
  // Check for validation errors
  const errors = validationResult(req);
  
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      errors: errors.array()
    });
  }
  
  // Process valid data
  const { email, password } = req.body;
  res.json({ 
    success: true, 
    message: 'User registered successfully',
    data: { email }
  });
});
```

### Middleware Pattern
```javascript
// middleware/validation.js
const { validationResult } = require('express-validator');

const handleValidationErrors = (req, res, next) => {
  const errors = validationResult(req);
  
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      message: 'Validation failed',
      errors: errors.array()
    });
  }
  
  next();
};

module.exports = { handleValidationErrors };

// Usage in routes
app.post('/register', userValidation, handleValidationErrors, (req, res) => {
  // Handle valid request
});
```

## Validation Methods

### Field Selection Methods
```javascript
const { body, param, query, header, cookie } = require('express-validator');

// Body field validation
body('username').isAlphanumeric()

// URL parameter validation
param('id').isInt()

// Query parameter validation
query('page').optional().isInt({ min: 1 })

// Header validation
header('authorization').exists()

// Cookie validation
cookie('sessionId').isLength({ min: 10 })
```

### Chaining Validations
```javascript
body('email')
  .exists().withMessage('Email is required')
  .isEmail().withMessage('Please provide a valid email')
  .normalizeEmail()
  .isLength({ max: 100 }).withMessage('Email must not exceed 100 characters')
```

### Optional vs Required Fields
```javascript
// Required field
body('name')
  .notEmpty()
  .withMessage('Name is required')

// Optional field (only validates if present)
body('age')
  .optional()
  .isInt({ min: 0, max: 120 })
  .withMessage('Age must be between 0 and 120')

// Optional field with default validation
body('country')
  .optional({ nullable: true, checkFalsy: true })
  .isLength({ min: 2, max: 2 })
  .withMessage('Country code must be 2 characters')
```

## Built-in Validators

### String Validators
```javascript
// Basic string validation
body('title')
  .isString()
  .withMessage('Title must be a string')
  .isLength({ min: 1, max: 100 })
  .withMessage('Title must be between 1 and 100 characters')
  .trim()

// Alphanumeric validation
body('username')
  .isAlphanumeric()
  .withMessage('Username must contain only letters and numbers')

// Alpha validation (letters only)
body('firstName')
  .isAlpha()
  .withMessage('First name must contain only letters')

// URL validation
body('website')
  .optional()
  .isURL({ protocols: ['http', 'https'] })
  .withMessage('Please provide a valid URL')
```

### Number Validators
```javascript
// Integer validation
body('age')
  .isInt({ min: 0, max: 150 })
  .withMessage('Age must be an integer between 0 and 150')

// Float validation
body('price')
  .isFloat({ min: 0 })
  .withMessage('Price must be a positive number')

// Numeric validation (string numbers)
body('phone')
  .isNumeric()
  .withMessage('Phone number must contain only numbers')
  .isLength({ min: 10, max: 15 })
  .withMessage('Phone number must be between 10 and 15 digits')
```

### Date Validators
```javascript
// ISO date validation
body('birthDate')
  .isISO8601()
  .withMessage('Please provide a valid date in ISO format')
  .toDate()

// Date range validation
body('eventDate')
  .isISO8601()
  .withMessage('Event date must be a valid date')
  .custom((value) => {
    if (new Date(value) < new Date()) {
      throw new Error('Event date must be in the future');
    }
    return true;
  })
```

### Email and Format Validators
```javascript
// Email validation
body('email')
  .isEmail()
  .withMessage('Please provide a valid email address')
  .normalizeEmail({
    gmail_remove_dots: false,
    gmail_remove_subaddress: false,
    outlookdotcom_remove_subaddress: false
  })

// UUID validation
body('userId')
  .isUUID(4)
  .withMessage('User ID must be a valid UUID v4')

// JSON validation
body('metadata')
  .optional()
  .isJSON()
  .withMessage('Metadata must be valid JSON')

// Credit card validation
body('cardNumber')
  .isCreditCard()
  .withMessage('Please provide a valid credit card number')
```

### Boolean and Choice Validators
```javascript
// Boolean validation
body('isActive')
  .isBoolean()
  .withMessage('isActive must be true or false')
  .toBoolean()

// Enum/Choice validation
body('status')
  .isIn(['active', 'inactive', 'pending'])
  .withMessage('Status must be one of: active, inactive, pending')

// Multiple choice validation
body('interests')
  .isArray({ min: 1, max: 5 })
  .withMessage('Please select 1-5 interests')
  .custom((interests) => {
    const validInterests = ['tech', 'sports', 'music', 'art', 'travel'];
    return interests.every(interest => validInterests.includes(interest));
  })
  .withMessage('Invalid interest selected')
```

### Pattern Matching
```javascript
// Regex validation
body('postalCode')
  .matches(/^\d{5}(-\d{4})?$/)
  .withMessage('Please provide a valid postal code (12345 or 12345-6789)')

// Strong password validation
body('password')
  .isLength({ min: 8 })
  .withMessage('Password must be at least 8 characters long')
  .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
  .withMessage('Password must contain at least one uppercase letter, one lowercase letter, one number, and one special character')
```

## Custom Validators

### Basic Custom Validator
```javascript
body('username')
  .custom(async (value) => {
    // Check if username already exists
    const existingUser = await User.findOne({ username: value });
    if (existingUser) {
      throw new Error('Username already exists');
    }
    return true;
  })
```

### Custom Validator with Database Check
```javascript
const { body } = require('express-validator');
const User = require('../models/User');

const checkEmailExists = async (email) => {
  const user = await User.findOne({ email });
  if (user) {
    throw new Error('Email already registered');
  }
  return true;
};

// Usage
body('email')
  .isEmail()
  .withMessage('Please provide a valid email')
  .custom(checkEmailExists)
```

### Custom Validator with Context
```javascript
body('confirmPassword')
  .custom((value, { req }) => {
    if (value !== req.body.password) {
      throw new Error('Password confirmation does not match password');
    }
    return true;
  })
```

### Async Custom Validator
```javascript
body('domain')
  .custom(async (value) => {
    try {
      const response = await fetch(`http://${value}`);
      if (!response.ok) {
        throw new Error('Domain is not accessible');
      }
      return true;
    } catch (error) {
      throw new Error('Invalid domain or domain is not accessible');
    }
  })
```

### Reusable Custom Validators
```javascript
// utils/customValidators.js
const User = require('../models/User');

const isUniqueEmail = async (email) => {
  const user = await User.findOne({ email });
  if (user) {
    throw new Error('Email already exists');
  }
  return true;
};

const isValidAge = (age) => {
  const numAge = parseInt(age);
  if (numAge < 13 || numAge > 120) {
    throw new Error('Age must be between 13 and 120');
  }
  return true;
};

const isStrongPassword = (password) => {
  const minLength = 8;
  const hasUpperCase = /[A-Z]/.test(password);
  const hasLowerCase = /[a-z]/.test(password);
  const hasNumbers = /\d/.test(password);
  const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);

  if (password.length < minLength) {
    throw new Error('Password must be at least 8 characters long');
  }
  
  if (!hasUpperCase || !hasLowerCase || !hasNumbers || !hasSpecialChar) {
    throw new Error('Password must contain uppercase, lowercase, number, and special character');
  }
  
  return true;
};

module.exports = {
  isUniqueEmail,
  isValidAge,
  isStrongPassword
};

// Usage
const { isUniqueEmail, isStrongPassword } = require('../utils/customValidators');

body('email').isEmail().custom(isUniqueEmail);
body('password').custom(isStrongPassword);
```

## Sanitization

### Built-in Sanitizers
```javascript
const { body } = require('express-validator');

// String sanitization
body('name')
  .trim()                    // Remove whitespace
  .escape()                  // HTML escape
  .toLowerCase()             // Convert to lowercase

body('email')
  .normalizeEmail()          // Normalize email format

// Number sanitization
body('age')
  .toInt()                   // Convert to integer

body('price')
  .toFloat()                 // Convert to float

// Boolean sanitization
body('isActive')
  .toBoolean()               // Convert to boolean

// Date sanitization
body('birthDate')
  .toDate()                  // Convert to Date object
```

### Advanced Sanitization
```javascript
// Custom sanitization
body('phone')
  .customSanitizer((value) => {
    // Remove all non-numeric characters
    return value.replace(/\D/g, '');
  })

// Blacklist characters
body('username')
  .blacklist('<>')           // Remove < and > characters

// Whitelist characters
body('alphanumericCode')
  .whitelist('a-zA-Z0-9')    // Keep only alphanumeric characters

// Limit string length
body('description')
  .trim()
  .customSanitizer((value) => {
    return value.length > 500 ? value.substring(0, 500) + '...' : value;
  })
```

### Sanitization Examples
```javascript
// Clean user input for database storage
const userSanitization = [
  body('firstName')
    .trim()
    .escape()
    .customSanitizer((value) => {
      // Capitalize first letter
      return value.charAt(0).toUpperCase() + value.slice(1).toLowerCase();
    }),
  
  body('bio')
    .trim()
    .customSanitizer((value) => {
      // Remove multiple spaces and line breaks
      return value.replace(/\s+/g, ' ').replace(/\n+/g, '\n');
    }),
  
  body('website')
    .optional()
    .customSanitizer((value) => {
      // Add protocol if missing
      if (value && !value.startsWith('http')) {
        return 'https://' + value;
      }
      return value;
    })
];
```

## Error Handling

### Basic Error Handling
```javascript
const { validationResult } = require('express-validator');

app.post('/api/users', userValidations, (req, res) => {
  const errors = validationResult(req);
  
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      errors: errors.array()
    });
  }
  
  // Process valid request
});
```

### Custom Error Formatting
```javascript
const formatErrors = (errors) => {
  const formattedErrors = {};
  
  errors.array().forEach(error => {
    if (!formattedErrors[error.path]) {
      formattedErrors[error.path] = [];
    }
    formattedErrors[error.path].push(error.msg);
  });
  
  return formattedErrors;
};

// Usage
const errors = validationResult(req);
if (!errors.isEmpty()) {
  return res.status(400).json({
    success: false,
    errors: formatErrors(errors)
  });
}
```

### Error Handling Middleware
```javascript
// middleware/errorHandler.js
const { validationResult } = require('express-validator');

const handleValidationErrors = (req, res, next) => {
  const errors = validationResult(req);
  
  if (!errors.isEmpty()) {
    const errorResponse = {
      success: false,
      message: 'Validation failed',
      errors: errors.array().map(error => ({
        field: error.path,
        message: error.msg,
        value: error.value
      }))
    };
    
    return res.status(400).json(errorResponse);
  }
  
  next();
};

// Advanced error handler with logging
const advancedErrorHandler = (req, res, next) => {
  const errors = validationResult(req);
  
  if (!errors.isEmpty()) {
    // Log validation errors
    console.error('Validation Error:', {
      url: req.url,
      method: req.method,
      errors: errors.array(),
      timestamp: new Date().toISOString()
    });
    
    const groupedErrors = {};
    errors.array().forEach(error => {
      if (!groupedErrors[error.path]) {
        groupedErrors[error.path] = [];
      }
      groupedErrors[error.path].push(error.msg);
    });
    
    return res.status(400).json({
      success: false,
      message: 'Validation failed',
      errors: groupedErrors,
      timestamp: new Date().toISOString()
    });
  }
  
  next();
};

module.exports = { handleValidationErrors, advancedErrorHandler };
```

### Error Localization
```javascript
// utils/errorMessages.js
const errorMessages = {
  en: {
    required: 'This field is required',
    email: 'Please provide a valid email address',
    minLength: 'Must be at least {min} characters long',
    maxLength: 'Must not exceed {max} characters'
  },
  es: {
    required: 'Este campo es obligatorio',
    email: 'Por favor proporcione una dirección de correo válida',
    minLength: 'Debe tener al menos {min} caracteres',
    maxLength: 'No debe exceder {max} caracteres'
  }
};

const getMessage = (key, lang = 'en', params = {}) => {
  let message = errorMessages[lang][key] || errorMessages.en[key];
  
  Object.keys(params).forEach(param => {
    message = message.replace(`{${param}}`, params[param]);
  });
  
  return message;
};

// Usage
body('name')
  .isLength({ min: 2, max: 50 })
  .withMessage((value, { req }) => {
    const lang = req.headers['accept-language'] || 'en';
    return getMessage('minLength', lang, { min: 2 });
  })
```

## Schema Validation

### Basic Schema
```javascript
const { checkSchema } = require('express-validator');

const userSchema = {
  email: {
    in: ['body'],
    isEmail: {
      errorMessage: 'Please provide a valid email'
    },
    normalizeEmail: true
  },
  password: {
    in: ['body'],
    isLength: {
      options: { min: 6 },
      errorMessage: 'Password must be at least 6 characters long'
    },
    matches: {
      options: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
      errorMessage: 'Password must contain uppercase, lowercase, and number'
    }
  },
  age: {
    in: ['body'],
    optional: true,
    isInt: {
      options: { min: 13, max: 120 },
      errorMessage: 'Age must be between 13 and 120'
    },
    toInt: true
  }
};

// Usage
app.post('/register', checkSchema(userSchema), handleValidationErrors, (req, res) => {
  // Handle valid request
});
```

### Advanced Schema with Custom Validators
```javascript
const userRegistrationSchema = {
  firstName: {
    in: ['body'],
    notEmpty: {
      errorMessage: 'First name is required'
    },
    isLength: {
      options: { min: 2, max: 50 },
      errorMessage: 'First name must be between 2 and 50 characters'
    },
    trim: true,
    escape: true
  },
  
  email: {
    in: ['body'],
    isEmail: {
      errorMessage: 'Please provide a valid email'
    },
    normalizeEmail: true,
    custom: {
      options: async (email) => {
        const existingUser = await User.findOne({ email });
        if (existingUser) {
          throw new Error('Email already registered');
        }
        return true;
      }
    }
  },
  
  password: {
    in: ['body'],
    isLength: {
      options: { min: 8 },
      errorMessage: 'Password must be at least 8 characters long'
    },
    custom: {
      options: (password) => {
        const strongPassword = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/;
        if (!strongPassword.test(password)) {
          throw new Error('Password must contain uppercase, lowercase, number, and special character');
        }
        return true;
      }
    }
  },
  
  confirmPassword: {
    in: ['body'],
    custom: {
      options: (confirmPassword, { req }) => {
        if (confirmPassword !== req.body.password) {
          throw new Error('Password confirmation does not match');
        }
        return true;
      }
    }
  },
  
  birthDate: {
    in: ['body'],
    optional: true,
    isISO8601: {
      errorMessage: 'Please provide a valid date'
    },
    custom: {
      options: (birthDate) => {
        const today = new Date();
        const birth = new Date(birthDate);
        const age = today.getFullYear() - birth.getFullYear();
        
        if (age < 13) {
          throw new Error('You must be at least 13 years old');
        }
        return true;
      }
    },
    toDate: true
  }
};
```

### Nested Object Validation
```javascript
const addressSchema = {
  'address.street': {
    in: ['body'],
    notEmpty: {
      errorMessage: 'Street address is required'
    },
    isLength: {
      options: { max: 100 },
      errorMessage: 'Street address must not exceed 100 characters'
    }
  },
  
  'address.city': {
    in: ['body'],
    notEmpty: {
      errorMessage: 'City is required'
    },
    isAlpha: {
      options: ['en-US', { ignore: ' -' }],
      errorMessage: 'City must contain only letters, spaces, and hyphens'
    }
  },
  
  'address.zipCode': {
    in: ['body'],
    matches: {
      options: /^\d{5}(-\d{4})?$/,
      errorMessage: 'Please provide a valid ZIP code'
    }
  },
  
  'address.country': {
    in: ['body'],
    isIn: {
      options: [['US', 'CA', 'MX', 'UK', 'DE', 'FR']],
      errorMessage: 'Please select a valid country'
    }
  }
};
```

### Array Validation in Schema
```javascript
const orderSchema = {
  'items.*.productId': {
    in: ['body'],
    isUUID: {
      options: 4,
      errorMessage: 'Product ID must be a valid UUID'
    }
  },
  
  'items.*.quantity': {
    in: ['body'],
    isInt: {
      options: { min: 1, max: 100 },
      errorMessage: 'Quantity must be between 1 and 100'
    },
    toInt: true
  },
  
  'items.*.price': {
    in: ['body'],
    isFloat: {
      options: { min: 0 },
      errorMessage: 'Price must be a positive number'
    },
    toFloat: true
  }
};
```

## Conditional Validation

### Basic Conditional Validation
```javascript
// Validate field only if another field has specific value
body('spouseName')
  .if(body('maritalStatus').equals('married'))
  .notEmpty()
  .withMessage('Spouse name is required for married individuals')

// Validate field only if it exists
body('website')
  .optional()
  .if((value) => value && value.length > 0)
  .isURL()
  .withMessage('Please provide a valid URL')
```

### Complex Conditional Logic
```javascript
const conditionalValidation = [
  // Age validation based on account type
  body('age')
    .if(body('accountType').equals('adult'))
    .isInt({ min: 18 })
    .withMessage('Adult accounts require age 18 or older'),
  
  body('age')
    .if(body('accountType').equals('minor'))
    .isInt({ min: 13, max: 17 })
    .withMessage('Minor accounts require age between 13 and 17'),
  
  // Company validation for business accounts
  body('companyName')
    .if(body('accountType').equals('business'))
    .notEmpty()
    .withMessage('Company name is required for business accounts'),
  
  body('taxId')
    .if(body('accountType').equals('business'))
    .matches(/^\d{2}-\d{7}$/)
    .withMessage('Please provide a valid tax ID (XX-XXXXXXX)')
];
```

### Conditional Validation with Custom Logic
```javascript
// Custom conditional validator
body('confirmEmail')
  .custom((value, { req }) => {
    // Only validate if email is being changed
    if (req.body.email && req.body.email !== req.user.email) {
      if (!value) {
        throw new Error('Email confirmation is required when changing email');
      }
      if (value !== req.body.email) {
        throw new Error('Email confirmation does not match');
      }
    }
    return true;
  })

// Validate payment method based on order total
body('paymentMethod')
  .custom((value, { req }) => {
    const total = parseFloat(req.body.total);
    
    if (total > 1000 && value === 'cash') {
      throw new Error('Cash payments not accepted for orders over $1000');
    }
    
    if (total > 5000 && !['wire_transfer', 'check'].includes(value)) {
      throw new Error('Large orders require wire transfer or check payment');
    }
    
    return true;
  })
```

### Role-based Conditional Validation
```javascript
// middleware/roleBasedValidation.js
const createRoleBasedValidation = (validations) => {
  return (req, res, next) => {
    const userRole = req.user?.role || 'guest';
    const roleValidations = validations[userRole] || validations.default || [];
    
    // Apply validations based on user role
    Promise.all(roleValidations.map(validation => validation.run(req)))
      .then(() => next())
      .catch(next);
  };
};

// Usage
const userUpdateValidations = {
  admin: [
    body('role').optional().isIn(['user', 'admin', 'moderator']),
    body('status').optional().isIn(['active', 'suspended', 'banned'])
  ],
  
  moderator: [
    body('status').optional().isIn(['active', 'suspended'])
  ],
  
  user: [
    body('name').optional().isLength({ min: 2, max: 50 }),
    body('email').optional().isEmail()
  ]
};

app.put('/users/:id', 
  authenticate, 
  createRoleBasedValidation(userUpdateValidations),
  handleValidationErrors,
  updateUser
);
```

## File Validation

### Basic File Validation with Multer
```javascript
const multer = require('multer');
const { body } = require('express-validator');

// Configure multer
const storage = multer.memoryStorage();
const upload = multer({
  storage,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB limit
    files: 5 // Maximum 5 files
  },
  fileFilter: (req, file, cb) => {
    // Allow only specific file types
    const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
    if (allowedTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type. Only JPEG, PNG, and GIF are allowed.'));
    }
  }
});

// File validation middleware
const validateFile = (req, res, next) => {
  if (!req.file && !req.files) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  const file = req.file || req.files[0];
  
  // Validate file size
  if (file.size > 5 * 1024 * 1024) {
    return res.status(400).json({ error: 'File size exceeds 5MB limit' });
  }
  
  // Validate file type
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
  if (!allowedTypes.includes(file.mimetype)) {
    return res.status(400).json({ error: 'Invalid file type' });
  }
  
  next();
};

// Route with file validation
app.post('/upload/avatar', 
  upload.single('avatar'),
  validateFile,
  (req, res) => {
    res.json({ 
      message: 'File uploaded successfully',
      file: {
        originalName: req.file.originalname,
        size: req.file.size,
        mimeType: req.file.mimetype
      }
    });
  }
);
```

### Advanced File Validation
```javascript
const sharp = require('sharp'); // For image processing

const validateImageFile = async (req, res, next) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'Image file is required' });
    }
    
    const { buffer, mimetype, size, originalname } = req.file;
    
    // Validate file extension matches MIME type
    const ext = originalname.split('.').pop().toLowerCase();
    const mimeTypeMap = {
      'jpg': 'image/jpeg',
      'jpeg': 'image/jpeg',
      'png': 'image/png',
      'gif': 'image/gif',
      'webp': 'image/webp'
    };
    
    if (mimeTypeMap[ext] !== mimetype) {
      return res.status(400).json({ 
        error: 'File extension does not match content type' 
      });
    }
    
    // Validate image dimensions and properties
    const metadata = await sharp(buffer).metadata();
    
    if (metadata.width > 4000 || metadata.height > 4000) {
      return res.status(400).json({ 
        error: 'Image dimensions exceed maximum (4000x4000)' 
      });
    }
    
    if (metadata.width < 100 || metadata.height < 100) {
      return res.status(400).json({ 
        error: 'Image dimensions below minimum (100x100)' 
      });
    }
    
    // Add metadata to request for further processing
    req.imageMetadata = metadata;
    next();
    
  } catch (error) {
    res.status(400).json({ 
      error: 'Invalid image file or corrupted data' 
    });
  }
};

// Multiple file validation
const validateMultipleFiles = (req, res, next) => {
  if (!req.files || req.files.length === 0) {
    return res.status(400).json({ error: 'At least one file is required' });
  }
  
  const errors = [];
  const maxFiles = 10;
  const maxTotalSize = 50 * 1024 * 1024; // 50MB total
  const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
  
  if (req.files.length > maxFiles) {
    errors.push(`Maximum ${maxFiles} files allowed`);
  }
  
  let totalSize = 0;
  req.files.forEach((file, index) => {
    totalSize += file.size;
    
    if (!allowedTypes.includes(file.mimetype)) {
      errors.push(`File ${index + 1}: Invalid file type`);
    }
    
    if (file.size > 10 * 1024 * 1024) { // 10MB per file
      errors.push(`File ${index + 1}: File size exceeds 10MB`);
    }
  });
  
  if (totalSize > maxTotalSize) {
    errors.push('Total file size exceeds 50MB limit');
  }
  
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }
  
  next();
};

// Document validation
const validateDocument = async (req, res, next) => {
  if (!req.file) {
    return res.status(400).json({ error: 'Document is required' });
  }
  
  const { buffer, mimetype, originalname, size } = req.file;
  
  // Validate document type
  const allowedTypes = {
    'application/pdf': '.pdf',
    'application/msword': '.doc',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document': '.docx',
    'text/plain': '.txt'
  };
  
  if (!allowedTypes[mimetype]) {
    return res.status(400).json({ 
      error: 'Invalid document type. Only PDF, DOC, DOCX, and TXT files are allowed' 
    });
  }
  
  // Validate file signature (magic numbers)
  const signatures = {
    pdf: [0x25, 0x50, 0x44, 0x46], // %PDF
    doc: [0xD0, 0xCF, 0x11, 0xE0], // MS Office
    txt: null // Text files don't have consistent signatures
  };
  
  if (mimetype === 'application/pdf') {
    const pdfSignature = Array.from(buffer.slice(0, 4));
    if (!signatures.pdf.every((byte, i) => byte === pdfSignature[i])) {
      return res.status(400).json({ 
        error: 'File appears to be corrupted or not a valid PDF' 
      });
    }
  }
  
  next();
};
```

### CSV File Validation
```javascript
const csv = require('csv-parser');
const { Readable } = require('stream');

const validateCSVFile = async (req, res, next) => {
  if (!req.file) {
    return res.status(400).json({ error: 'CSV file is required' });
  }
  
  const { buffer, mimetype } = req.file;
  
  if (mimetype !== 'text/csv' && !req.file.originalname.endsWith('.csv')) {
    return res.status(400).json({ error: 'File must be a CSV' });
  }
  
  try {
    const rows = [];
    const errors = [];
    let rowCount = 0;
    const maxRows = 10000;
    
    await new Promise((resolve, reject) => {
      const stream = Readable.from(buffer.toString());
      
      stream
        .pipe(csv())
        .on('data', (row) => {
          rowCount++;
          
          if (rowCount > maxRows) {
            reject(new Error(`CSV exceeds maximum of ${maxRows} rows`));
            return;
          }
          
          // Validate required columns
          const requiredColumns = ['name', 'email', 'age'];
          const missingColumns = requiredColumns.filter(col => !row[col]);
          
          if (missingColumns.length > 0) {
            errors.push(`Row ${rowCount}: Missing columns: ${missingColumns.join(', ')}`);
          }
          
          // Validate email format
          if (row.email && !/\S+@\S+\.\S+/.test(row.email)) {
            errors.push(`Row ${rowCount}: Invalid email format`);
          }
          
          // Validate age
          if (row.age && (isNaN(row.age) || row.age < 0 || row.age > 120)) {
            errors.push(`Row ${rowCount}: Invalid age`);
          }
          
          rows.push(row);
        })
        .on('end', () => {
          if (errors.length > 10) {
            reject(new Error(`Too many validation errors (${errors.length}). Please fix the CSV and try again.`));
          } else if (errors.length > 0) {
            reject(new Error(`CSV validation errors:\n${errors.slice(0, 10).join('\n')}`));
          } else {
            resolve();
          }
        })
        .on('error', reject);
    });
    
    req.csvData = rows;
    next();
    
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};
```

## Advanced Techniques

### Custom Validation Chain
```javascript
// utils/validationChains.js
const { body } = require('express-validator');

const createPasswordValidation = (fieldName = 'password') => {
  return body(fieldName)
    .isLength({ min: 8, max: 128 })
    .withMessage('Password must be between 8 and 128 characters')
    .matches(/^(?=.*[a-z])/)
    .withMessage('Password must contain at least one lowercase letter')
    .matches(/^(?=.*[A-Z])/)
    .withMessage('Password must contain at least one uppercase letter')
    .matches(/^(?=.*\d)/)
    .withMessage('Password must contain at least one number')
    .matches(/^(?=.*[@$!%*?&])/)
    .withMessage('Password must contain at least one special character')
    .not()
    .matches(/^(.)\1+$/)
    .withMessage('Password cannot be all the same character');
};

const createEmailValidation = (fieldName = 'email', options = {}) => {
  let validation = body(fieldName)
    .isEmail()
    .withMessage('Please provide a valid email address')
    .normalizeEmail();
  
  if (options.checkUnique) {
    validation = validation.custom(async (email) => {
      const existingUser = await User.findOne({ email });
      if (existingUser) {
        throw new Error('Email already registered');
      }
      return true;
    });
  }
  
  if (options.domains) {
    validation = validation.custom((email) => {
      const domain = email.split('@')[1];
      if (!options.domains.includes(domain)) {
        throw new Error(`Email domain must be one of: ${options.domains.join(', ')}`);
      }
      return true;
    });
  }
  
  return validation;
};

module.exports = {
  createPasswordValidation,
  createEmailValidation
};

// Usage
const { createPasswordValidation, createEmailValidation } = require('../utils/validationChains');

const registrationValidation = [
  createEmailValidation('email', { checkUnique: true }),
  createPasswordValidation('password'),
  body('confirmPassword')
    .custom((value, { req }) => {
      if (value !== req.body.password) {
        throw new Error('Password confirmation does not match');
      }
      return true;
    })
];
```

### Validation Composition
```javascript
// Higher-order validation functions
const withOptional = (validation) => {
  return validation.optional({ nullable: true, checkFalsy: true });
};

const withTrim = (validation) => {
  return validation.trim();
};

const withEscape = (validation) => {
  return validation.escape();
};

// Compose validations
const createStringValidation = (fieldName, options = {}) => {
  let validation = body(fieldName);
  
  if (options.optional) {
    validation = withOptional(validation);
  } else {
    validation = validation.notEmpty().withMessage(`${fieldName} is required`);
  }
  
  if (options.trim) {
    validation = withTrim(validation);
  }
  
  if (options.escape) {
    validation = withEscape(validation);
  }
  
  if (options.minLength || options.maxLength) {
    validation = validation.isLength({
      min: options.minLength || 0,
      max: options.maxLength || Infinity
    }).withMessage(`${fieldName} must be between ${options.minLength || 0} and ${options.maxLength || 'unlimited'} characters`);
  }
  
  return validation;
};

// Usage
const userValidation = [
  createStringValidation('firstName', { trim: true, escape: true, minLength: 2, maxLength: 50 }),
  createStringValidation('lastName', { trim: true, escape: true, minLength: 2, maxLength: 50 }),
  createStringValidation('bio', { optional: true, trim: true, maxLength: 500 })
];
```

### Dynamic Validation
```javascript
// Dynamic validation based on request data
const createDynamicValidation = (req) => {
  const validations = [];
  
  // Base validations
  validations.push(
    body('name').notEmpty().withMessage('Name is required'),
    body('email').isEmail().withMessage('Valid email required')
  );
  
  // Add validations based on user type
  if (req.body.userType === 'business') {
    validations.push(
      body('companyName').notEmpty().withMessage('Company name required for business accounts'),
      body('taxId').matches(/^\d{2}-\d{7}$/).withMessage('Valid tax ID required')
    );
  }
  
  // Add validations based on subscription level
  if (req.body.subscriptionLevel === 'premium') {
    validations.push(
      body('paymentMethod').isIn(['credit_card', 'paypal']).withMessage('Payment method required for premium')
    );
  }
  
  return validations;
};

// Middleware to apply dynamic validations
const applyDynamicValidation = async (req, res, next) => {
  const validations = createDynamicValidation(req);
  
  // Run all validations
  await Promise.all(validations.map(validation => validation.run(req)));
  
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  
  next();
};
```

### Validation Caching
```javascript
// Cache validation results for expensive operations
const NodeCache = require('node-cache');
const validationCache = new NodeCache({ stdTTL: 300 }); // 5 minutes

const cachedEmailValidation = body('email')
  .isEmail()
  .custom(async (email) => {
    const cacheKey = `email_exists_${email}`;
    
    // Check cache first
    const cached = validationCache.get(cacheKey);
    if (cached !== undefined) {
      if (cached) {
        throw new Error('Email already exists');
      }
      return true;
    }
    
    // Check database
    const existingUser = await User.findOne({ email });
    const exists = !!existingUser;
    
    // Cache result
    validationCache.set(cacheKey, exists);
    
    if (exists) {
      throw new Error('Email already exists');
    }
    
    return true;
  });
```

## Testing Validation

### Unit Testing Validators
```javascript
// tests/validators.test.js
const request = require('supertest');
const express = require('express');
const { body, validationResult } = require('express-validator');

const app = express();
app.use(express.json());

// Test route
app.post('/test', [
  body('email').isEmail().withMessage('Invalid email'),
  body('age').isInt({ min: 0 }).withMessage('Age must be a positive integer')
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  res.json({ success: true });
});

describe('Validation Tests', () => {
  describe('Email Validation', () => {
    it('should accept valid email', async () => {
      const response = await request(app)
        .post('/test')
        .send({ email: 'test@example.com', age: 25 });
      
      expect(response.status).toBe(200);
      expect(response.body.success).toBe(true);
    });
    
    it('should reject invalid email', async () => {
      const response = await request(app)
        .post('/test')
        .send({ email: 'invalid-email', age: 25 });
      
      expect(response.status).toBe(400);
      expect(response.body.errors).toHaveLength(1);
      expect(response.body.errors[0].msg).toBe('Invalid email');
    });
  });
  
  describe('Age Validation', () => {
    it('should accept valid age', async () => {
      const response = await request(app)
        .post('/test')
        .send({ email: 'test@example.com', age: 25 });
      
      expect(response.status).toBe(200);
    });
    
    it('should reject negative age', async () => {
      const response = await request(app)
        .post('/test')
        .send({ email: 'test@example.com', age: -5 });
      
      expect(response.status).toBe(400);
      expect(response.body.errors[0].msg).toBe('Age must be a positive integer');
    });
  });
});
```

### Integration Testing
```javascript
// tests/integration/user.test.js
describe('User Registration Integration', () => {
  beforeEach(async () => {
    // Clear database
    await User.deleteMany({});
  });
  
  it('should register user with valid data', async () => {
    const userData = {
      firstName: 'John',
      lastName: 'Doe',
      email: 'john@example.com',
      password: 'Password123!',
      confirmPassword: 'Password123!',
      age: 25
    };
    
    const response = await request(app)
      .post('/api/users/register')
      .send(userData);
    
    expect(response.status).toBe(201);
    expect(response.body.success).toBe(true);
    
    // Verify user was created in database
    const user = await User.findOne({ email: userData.email });
    expect(user).toBeTruthy();
    expect(user.firstName).toBe(userData.firstName);
  });
  
  it('should prevent duplicate email registration', async () => {
    // Create existing user
    await User.create({
      firstName: 'Jane',
      lastName: 'Doe',
      email: 'jane@example.com',
      password: 'hashedpassword'
    });
    
    const duplicateUserData = {
      firstName: 'John',
      lastName: 'Smith',
      email: 'jane@example.com',
      password: 'Password123!',
      confirmPassword: 'Password123!'
    };
    
    const response = await request(app)
      .post('/api/users/register')
      .send(duplicateUserData);
    
    expect(response.status).toBe(400);
    expect(response.body.errors).toContainEqual(
      expect.objectContaining({
        msg: 'Email already registered'
      })
    );
  });
});
```

### Testing Custom Validators
```javascript
// tests/customValidators.test.js
const { isUniqueEmail, isStrongPassword } = require('../utils/customValidators');

describe('Custom Validators', () => {
  describe('isUniqueEmail', () => {
    beforeEach(async () => {
      await User.deleteMany({});
    });
    
    it('should pass for unique email', async () => {
      await expect(isUniqueEmail('unique@example.com')).resolves.toBe(true);
    });
    
    it('should throw error for existing email', async () => {
      await User.create({ 
        email: 'existing@example.com', 
        password: 'hashedpassword' 
      });
      
      await expect(isUniqueEmail('existing@example.com'))
        .rejects.toThrow('Email already exists');
    });
  });
  
  describe('isStrongPassword', () => {
    it('should pass for strong password', () => {
      expect(() => isStrongPassword('StrongPass123!')).not.toThrow();
    });
    
    it('should throw error for weak password', () => {
      expect(() => isStrongPassword('weak')).toThrow();
    });
    
    it('should throw error for password without uppercase', () => {
      expect(() => isStrongPassword('password123!'))
        .toThrow('Password must contain uppercase, lowercase, number, and special character');
    });
  });
});
```

## Best Practices

### 1. Organization and Structure
```javascript
// Good: Organized validation files
// validators/userValidators.js
const { body } = require('express-validator');
const { isUniqueEmail } = require('../utils/customValidators');

const registrationValidation = [
  body('email').isEmail().custom(isUniqueEmail),
  body('password').isLength({ min: 8 }).matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
];

const updateValidation = [
  body('firstName').optional().trim().isLength({ min: 2, max: 50 }),
  body('lastName').optional().trim().isLength({ min: 2, max: 50 })
];

module.exports = { registrationValidation, updateValidation };

// routes/users.js
const { registrationValidation } = require('../validators/userValidators');
const { handleValidationErrors } = require('../middleware/validation');

router.post('/register', registrationValidation, handleValidationErrors, registerUser);
```

### 2. Error Message Consistency
```javascript
// utils/errorMessages.js
const ERROR_MESSAGES = {
  REQUIRED: (field) => `${field} is required`,
  INVALID_EMAIL: 'Please provide a valid email address',
  INVALID_PASSWORD: 'Password must contain uppercase, lowercase, number, and special character',
  MIN_LENGTH: (field, min) => `${field} must be at least ${min} characters long`,
  MAX_LENGTH: (field, max) => `${field} must not exceed ${max} characters`,
  UNIQUE_EMAIL: 'Email address is already registered',
  PASSWORD_MISMATCH: 'Password confirmation does not match password'
};

// Usage
body('email')
  .isEmail()
  .withMessage(ERROR_MESSAGES.INVALID_EMAIL)
  .custom(isUniqueEmail)
  .withMessage(ERROR_MESSAGES.UNIQUE_EMAIL);
```

### 3. Reusable Validation Chains
```javascript
// utils/commonValidations.js
const createRequiredStringValidation = (fieldName, minLength = 1, maxLength = 255) => {
  return body(fieldName)
    .trim()
    .notEmpty()
    .withMessage(`${fieldName} is required`)
    .isLength({ min: minLength, max: maxLength })
    .withMessage(`${fieldName} must be between ${minLength} and ${maxLength} characters`)
    .escape();
};

const createOptionalStringValidation = (fieldName, maxLength = 255) => {
  return body(fieldName)
    .optional({ nullable: true, checkFalsy: true })
    .trim()
    .isLength({ max: maxLength })
    .withMessage(`${fieldName} must not exceed ${maxLength} characters`)
    .escape();
};

const createEmailValidation = (fieldName = 'email', checkUnique = false) => {
  let validation = body(fieldName)
    .isEmail()
    .withMessage('Please provide a valid email address')
    .normalizeEmail();
  
  if (checkUnique) {
    validation = validation.custom(isUniqueEmail);
  }
  
  return validation;
};
```

### 4. Performance Optimization
```javascript
// Use optional validation wisely
body('optionalField')
  .optional({ nullable: true, checkFalsy: true }) // Skip validation if null, undefined, or empty string
  .isEmail();

// Avoid expensive operations in validation
body('email')
  .isEmail() // Fast validation first
  .custom(async (email) => {
    // Expensive database check only if email format is valid
    const exists = await User.findOne({ email });
    if (exists) throw new Error('Email exists');
    return true;
  });

// Use bail() to stop validation chain on first error
body('password')
  .notEmpty().withMessage('Password is required').bail()
  .isLength({ min: 8 }).withMessage('Password too short').bail()
  .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
  .withMessage('Password must contain uppercase, lowercase, and number');
```

### 5. Security Best Practices
```javascript
// Always sanitize input
body('userInput')
  .trim()           // Remove whitespace
  .escape()         // HTML escape
  .blacklist('<>')  // Remove specific characters

// Validate file uploads carefully
const validateFileUpload = (req, res, next) => {
  if (!req.file) {
    return res.status(400).json({ error: 'File is required' });
  }
  
  // Check file size
  if (req.file.size > 5 * 1024 * 1024) {
    return res.status(400).json({ error: 'File too large' });
  }
  
  // Check MIME type AND file extension
  const allowedTypes = ['image/jpeg', 'image/png'];
  const allowedExtensions = ['.jpg', '.jpeg', '.png'];
  
  if (!allowedTypes.includes(req.file.mimetype)) {
    return res.status(400).json({ error: 'Invalid file type' });
  }
  
  const ext = path.extname(req.file.originalname).toLowerCase();
  if (!allowedExtensions.includes(ext)) {
    return res.status(400).json({ error: 'Invalid file extension' });
  }
  
  next();
};

// Rate limiting for validation-heavy endpoints
const rateLimit = require('express-rate-limit');

const validationRateLimit = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many validation requests from this IP'
});

app.use('/api/validate', validationRateLimit);
```

## Common Patterns

### 1. User Registration Pattern
```javascript
// validators/authValidators.js
const registrationValidation = [
  body('firstName')
    .trim()
    .notEmpty()
    .withMessage('First name is required')
    .isLength({ min: 2, max: 50 })
    .withMessage('First name must be between 2 and 50 characters')
    .matches(/^[a-zA-Z\s'-]+$/)
    .withMessage('First name can only contain letters, spaces, hyphens, and apostrophes'),
  
  body('lastName')
    .trim()
    .notEmpty()
    .withMessage('Last name is required')
    .isLength({ min: 2, max: 50 })
    .withMessage('Last name must be between 2 and 50 characters')
    .matches(/^[a-zA-Z\s'-]+$/)
    .withMessage('Last name can only contain letters, spaces, hyphens, and apostrophes'),
  
  body('email')
    .isEmail()
    .withMessage('Please provide a valid email address')
    .normalizeEmail()
    .custom(async (email) => {
      const existingUser = await User.findOne({ email });
      if (existingUser) {
        throw new Error('Email address is already registered');
      }
      return true;
    }),
  
  body('password')
    .isLength({ min: 8, max: 128 })
    .withMessage('Password must be between 8 and 128 characters')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
    .withMessage('Password must contain at least one uppercase letter, lowercase letter, number, and special character'),
  
  body('confirmPassword')
    .custom((value, { req }) => {
      if (value !== req.body.password) {
        throw new Error('Password confirmation does not match password');
      }
      return true;
    }),
  
  body('dateOfBirth')
    .optional()
    .isISO8601()
    .withMessage('Please provide a valid date of birth')
    .custom((value) => {
      const age = (new Date() - new Date(value)) / (1000 * 60 * 60 * 24 * 365);
      if (age < 13) {
        throw new Error('You must be at least 13 years old to register');
      }
      return true;
    })
    .toDate(),
  
  body('termsAccepted')
    .equals('true')
    .withMessage('You must accept the terms and conditions')
    .toBoolean()
];
```

### 2. E-commerce Product Pattern
```javascript
// validators/productValidators.js
const productValidation = [
  body('name')
    .trim()
    .notEmpty()
    .withMessage('Product name is required')
    .isLength({ min: 3, max: 100 })
    .withMessage('Product name must be between 3 and 100 characters'),
  
  body('description')
    .trim()
    .notEmpty()
    .withMessage('Product description is required')
    .isLength({ min: 10, max: 2000 })
    .withMessage('Description must be between 10 and 2000 characters'),
  
  body('price')
    .isFloat({ min: 0.01, max: 999999.99 })
    .withMessage('Price must be between 0.01 and 999999.99')
    .toFloat(),
  
  body('category')
    .notEmpty()
    .withMessage('Category is required')
    .isIn(['electronics', 'clothing', 'books', 'home', 'sports'])
    .withMessage('Invalid category selected'),
  
  body('tags')
    .optional()
    .isArray({ min: 0, max: 10 })
    .withMessage('Maximum 10 tags allowed')
    .custom((tags) => {
      if (tags.some(tag => typeof tag !== 'string' || tag.length > 20)) {
        throw new Error('Each tag must be a string with maximum 20 characters');
      }
      return true;
    }),
  
  body('inventory.quantity')
    .isInt({ min: 0 })
    .withMessage('Inventory quantity must be a non-negative integer')
    .toInt(),
  
  body('inventory.sku')
    .optional()
    .matches(/^[A-Z0-9-]+$/)
    .withMessage('SKU can only contain uppercase letters, numbers, and hyphens')
    .isLength({ max: 20 })
    .withMessage('SKU must not exceed 20 characters'),
  
  body('dimensions.weight')
    .optional()
    .isFloat({ min: 0 })
    .withMessage('Weight must be a positive number')
    .toFloat(),
  
  body('dimensions.length')
    .optional()
    .isFloat({ min: 0 })
    .withMessage('Length must be a positive number')
    .toFloat(),
  
  body('dimensions.width')
    .optional()
    .isFloat({ min: 0 })
    .withMessage('Width must be a positive number')
    .toFloat(),
  
  body('dimensions.height')
    .optional()
    .isFloat({ min: 0 })
    .withMessage('Height must be a positive number')
    .toFloat()
];
```

### 3. API Pagination Pattern
```javascript
// validators/paginationValidators.js
const paginationValidation = [
  query('page')
    .optional()
    .isInt({ min: 1 })
    .withMessage('Page must be a positive integer')
    .toInt(),
  
  query('limit')
    .optional()
    .isInt({ min: 1, max: 100 })
    .withMessage('Limit must be between 1 and 100')
    .toInt(),
  
  query('sort')
    .optional()
    .isIn(['name', 'date', 'price', 'rating', '-name', '-date', '-price', '-rating'])
    .withMessage('Invalid sort parameter'),
  
  query('search')
    .optional()
    .trim()
    .isLength({ min: 1, max: 100 })
    .withMessage('Search query must be between 1 and 100 characters')
    .escape()
];
```

### 4. Contact Form Pattern
```javascript
// validators/contactValidators.js
const contactFormValidation = [
  body('name')
    .trim()
    .notEmpty()
    .withMessage('Name is required')
    .isLength({ min: 2, max: 100 })
    .withMessage('Name must be between 2 and 100 characters')
    .matches(/^[a-zA-Z\s'-]+$/)
    .withMessage('Name can only contain letters, spaces, hyphens, and apostrophes'),
  
  body('email')
    .isEmail()
    .withMessage('Please provide a valid email address')
    .normalizeEmail(),
  
  body('phone')
    .optional()
    .matches(/^[\+]?[1-9][\d]{0,15}$/)
    .withMessage('Please provide a valid phone number'),
  
  body('subject')
    .trim()
    .notEmpty()
    .withMessage('Subject is required')
    .isLength({ min: 5, max: 200 })
    .withMessage('Subject must be between 5 and 200 characters'),
  
  body('message')
    .trim()
    .notEmpty()
    .withMessage('Message is required')
    .isLength({ min: 10, max: 2000 })
    .withMessage('Message must be between 10 and 2000 characters'),
  
  body('category')
    .isIn(['general', 'support', 'sales', 'feedback', 'bug_report'])
    .withMessage('Please select a valid category'),
  
  body('priority')
    .optional()
    .isIn(['low', 'medium', 'high'])
    .withMessage('Invalid priority level')
];
```

### 5. Search and Filter Pattern
```javascript
// validators/searchValidators.js
const searchValidation = [
  query('q')
    .optional()
    .trim()
    .isLength({ min: 1, max: 200 })
    .withMessage('Search query must be between 1 and 200 characters')
    .escape(),
  
  query('category')
    .optional()
    .isArray()
    .custom((categories) => {
      const validCategories = ['electronics', 'clothing', 'books', 'home'];
      return categories.every(cat => validCategories.includes(cat));
    })
    .withMessage('Invalid category selected'),
  
  query('priceMin')
    .optional()
    .isFloat({ min: 0 })
    .withMessage('Minimum price must be a positive number')
    .toFloat(),
  
  query('priceMax')
    .optional()
    .isFloat({ min: 0 })
    .withMessage('Maximum price must be a positive number')
    .toFloat()
    .custom((priceMax, { req }) => {
      if (req.query.priceMin && parseFloat(priceMax) < parseFloat(req.query.priceMin)) {
        throw new Error('Maximum price must be greater than minimum price');
      }
      return true;
    }),
  
  query('rating')
    .optional()
    .isFloat({ min: 1, max: 5 })
    .withMessage('Rating must be between 1 and 5')
    .toFloat(),
  
  query('inStock')
    .optional()
    .isBoolean()
    .withMessage('inStock must be true or false')
    .toBoolean(),
  
  query('brand')
    .optional()
    .isArray()
    .custom((brands) => {
      return brands.every(brand => typeof brand === 'string' && brand.length <= 50);
    })
    .withMessage('Invalid brand filter')
];
```

## Performance Considerations

### 1. Validation Order Optimization
```javascript
// Optimize validation order: fast validations first, expensive ones last
const optimizedUserValidation = [
  // Fast validations first
  body('email')
    .notEmpty().withMessage('Email is required').bail()
    .isEmail().withMessage('Invalid email format').bail()
    .normalizeEmail()
    // Expensive database check last
    .custom(async (email) => {
      const exists = await User.findOne({ email });
      if (exists) throw new Error('Email already exists');
      return true;
    }),
  
  body('password')
    .notEmpty().withMessage('Password is required').bail()
    .isLength({ min: 8 }).withMessage('Password too short').bail()
    // Complex regex validation after basic checks
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/)
    .withMessage('Password must meet complexity requirements')
];
```

### 2. Caching Expensive Validations
```javascript
const NodeCache = require('node-cache');
const validationCache = new NodeCache({ stdTTL: 600 }); // 10 minutes

const cachedUniqueValidation = (Model, field) => {
  return async (value) => {
    const cacheKey = `${Model.modelName}_${field}_${value}`;
    
    // Check cache first
    const cached = validationCache.get(cacheKey);
    if (cached !== undefined) {
      if (cached) throw new Error(`${field} already exists`);
      return true;
    }
    
    // Query database
    const exists = await Model.findOne({ [field]: value });
    const result = !!exists;
    
    // Cache result
    validationCache.set(cacheKey, result);
    
    if (result) throw new Error(`${field} already exists`);
    return true;
  };
};

// Usage
body('email')
  .isEmail()
  .custom(cachedUniqueValidation(User, 'email'));
```

### 3. Conditional Validation Loading
```javascript
// Load validations only when needed
const getValidationRules = (userType, action) => {
  const baseValidations = [
    body('name').notEmpty().withMessage('Name is required')
  ];
  
  if (action === 'create') {
    baseValidations.push(
      body('email').isEmail().custom(checkUniqueEmail)
    );
  }
  
  if (userType === 'admin') {
    baseValidations.push(
      body('permissions').isArray().withMessage('Permissions must be an array')
    );
  }
  
  return baseValidations;
};

// Dynamic middleware
const dynamicValidation = (req, res, next) => {
  const userType = req.user?.type || 'regular';
  const action = req.method === 'POST' ? 'create' : 'update';
  
  const validations = getValidationRules(userType, action);
  
  Promise.all(validations.map(validation => validation.run(req)))
    .then(() => {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return res.status(400).json({ errors: errors.array() });
      }
      next();
    })
    .catch(next);
};
```

### 4. Batch Validation Optimization
```javascript
// Optimize for bulk operations
const bulkUserValidation = [
  body('users')
    .isArray({ min: 1, max: 1000 })
    .withMessage('Users array must contain 1-1000 items'),
  
  body('users.*.email')
    .isEmail()
    .withMessage('Each user must have a valid email'),
  
  body('users.*.name')
    .trim()
    .isLength({ min: 2, max: 100 })
    .withMessage('Each user name must be 2-100 characters'),
  
  // Custom validation for duplicate emails within the batch
  body('users')
    .custom((users) => {
      const emails = users.map(user => user.email);
      const uniqueEmails = new Set(emails);
      
      if (emails.length !== uniqueEmails.size) {
        throw new Error('Duplicate emails found in the batch');
      }
      
      return true;
    })
];

// Optimized database check for bulk emails
const checkBulkEmailUniqueness = async (req, res, next) => {
  const emails = req.body.users.map(user => user.email);
  
  const existingUsers = await User.find({ 
    email: { $in: emails } 
  }).select('email');
  
  if (existingUsers.length > 0) {
    const existingEmails = existingUsers.map(user => user.email);
    return res.status(400).json({
      error: 'Some emails already exist',
      existingEmails
    });
  }
  
  next();
};
```

## Integration Examples

### 1. MongoDB Integration
```javascript
// models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  email: { type: String, unique: true, required: true },
  username: { type: String, unique: true, required: true },
  password: { type: String, required: true },
  profile: {
    firstName: String,
    lastName: String,
    dateOfBirth: Date
  }
}, { timestamps: true });

const User = mongoose.model('User', userSchema);

// validators/userValidators.js with MongoDB integration
const { body } = require('express-validator');

const checkUniqueEmail = async (email, { req }) => {
  const existingUser = await User.findOne({ email });
  
  // For updates, exclude current user
  if (existingUser && existingUser._id.toString() !== req.params.id) {
    throw new Error('Email already exists');
  }
  
  return true;
};

const checkUniqueUsername = async (username, { req }) => {
  const existingUser = await User.findOne({ username });
  
  if (existingUser && existingUser._id.toString() !== req.params.id) {
    throw new Error('Username already exists');
  }
  
  return true;
};

const userCreateValidation = [
  body('email')
    .isEmail()
    .withMessage('Please provide a valid email')
    .normalizeEmail()
    .custom(checkUniqueEmail),
  
  body('username')
    .isLength({ min: 3, max: 30 })
    .withMessage('Username must be 3-30 characters')
    .matches(/^[a-zA-Z0-9_]+$/)
    .withMessage('Username can only contain letters, numbers, and underscores')
    .custom(checkUniqueUsername),
  
  body('profile.firstName')
    .optional()
    .trim()
    .isLength({ min: 2, max: 50 })
    .withMessage('First name must be 2-50 characters'),
  
  body('profile.dateOfBirth')
    .optional()
    .isISO8601()
    .withMessage('Please provide a valid date of birth')
    .custom((value) => {
      const age = (Date.now() - new Date(value).getTime()) / (1000 * 60 * 60 * 24 * 365);
      if (age < 13) throw new Error('Must be at least 13 years old');
      return true;
    })
    .toDate()
];
```

### 2. PostgreSQL with Sequelize Integration
```javascript
// models/Product.js
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');

const Product = sequelize.define('Product', {
  name: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true
  },
  sku: {
    type: DataTypes.STRING,
    unique: true
  },
  price: {
    type: DataTypes.DECIMAL(10, 2),
    allowNull: false
  },
  categoryId: {
    type: DataTypes.INTEGER,
    references: {
      model: 'Categories',
      key: 'id'
    }
  }
});

// validators/productValidators.js with Sequelize integration
const checkUniqueSKU = async (sku, { req }) => {
  const existingProduct = await Product.findOne({ where: { sku } });
  
  if (existingProduct && existingProduct.id !== parseInt(req.params.id)) {
    throw new Error('SKU already exists');
  }
  
  return true;
};

const checkCategoryExists = async (categoryId) => {
  const category = await Category.findByPk(categoryId);
  if (!category) {
    throw new Error('Category does not exist');
  }
  return true;
};

const productValidation = [
  body('name')
    .trim()
    .notEmpty()
    .withMessage('Product name is required')
    .isLength({ max: 255 })
    .withMessage('Product name too long'),
  
  body('sku')
    .optional()
    .trim()
    .matches(/^[A-Z0-9-]+$/)
    .withMessage('SKU can only contain uppercase letters, numbers, and hyphens')
    .custom(checkUniqueSKU),
  
  body('price')
    .isFloat({ min: 0.01 })
    .withMessage('Price must be greater than 0')
    .toFloat(),
  
  body('categoryId')
    .isInt()
    .withMessage('Category ID must be an integer')
    .custom(checkCategoryExists)
    .toInt()
];
```

### 3. JWT Authentication Integration
```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');

const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }
  
  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid token' });
    }
    req.user = user;
    next();
  });
};

// validators/authValidators.js with JWT integration
const validateCurrentPassword = async (currentPassword, { req }) => {
  if (!req.user) {
    throw new Error('Authentication required');
  }
  
  const user = await User.findById(req.user.id);
  const isValid = await bcrypt.compare(currentPassword, user.password);
  
  if (!isValid) {
    throw new Error('Current password is incorrect');
  }
  
  return true;
};

const passwordChangeValidation = [
  body('currentPassword')
    .notEmpty()
    .withMessage('Current password is required')
    .custom(validateCurrentPassword),
  
  body('newPassword')
    .isLength({ min: 8 })
    .withMessage('New password must be at least 8 characters')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/)
    .withMessage('New password must meet complexity requirements'),
  
  body('confirmNewPassword')
    .custom((value, { req }) => {
      if (value !== req.body.newPassword) {
        throw new Error('Password confirmation does not match');
      }
      return true;
    })
];

// Usage
app.put('/api/users/change-password',
  authenticateToken,
  passwordChangeValidation,
  handleValidationErrors,
  changePassword
);
```

### 4. File Upload with Multer Integration
```javascript
// config/multer.js
const multer = require('multer');
const path = require('path');

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});

const upload = multer({
  storage,
  limits: {
    fileSize: 10 * 1024 * 1024, // 10MB
    files: 5
  },
  fileFilter: (req, file, cb) => {
    const allowedTypes = /jpeg|jpg|png|gif/;
    const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
    const mimetype = allowedTypes.test(file.mimetype);
    
    if (mimetype && extname) {
      return cb(null, true);
    } else {
      cb(new Error('Only image files are allowed'));
    }
  }
});

// validators/uploadValidators.js
const { body } = require('express-validator');

const validateFileUpload = (req, res, next) => {
  if (!req.files || req.files.length === 0) {
    return res.status(400).json({ error: 'At least one file is required' });
  }
  
  const errors = [];
  
  req.files.forEach((file, index) => {
    // Validate file size
    if (file.size > 5 * 1024 * 1024) {
      errors.push(`File ${index + 1}: Size exceeds 5MB limit`);
    }
    
    // Validate dimensions (if image)
    if (file.mimetype.startsWith('image/')) {
      // This would require image processing library like sharp
      // Implementation depends on specific requirements
    }
  });
  
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }
  
  next();
};

const galleryUploadValidation = [
  body('title')
    .trim()
    .notEmpty()
    .withMessage('Gallery title is required')
    .isLength({ max: 100 })
    .withMessage('Title must not exceed 100 characters'),
  
  body('description')
    .optional()
    .trim()
    .isLength({ max: 500 })
    .withMessage('Description must not exceed 500 characters'),
  
  body('tags')
    .optional()
    .isArray({ max: 10 })
    .withMessage('Maximum 10 tags allowed')
];

// Route with file upload validation
app.post('/api/gallery',
  upload.array('images', 5),
  validateFileUpload,
  galleryUploadValidation,
  handleValidationErrors,
  createGallery
);
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Validation Not Running
```javascript
// Problem: Validation rules not being applied
// Solution: Ensure middleware is properly chained

// Wrong ❌
app.post('/api/users', (req, res) => {
  // Validation never runs
  body('email').isEmail();
  // ... rest of handler
});

// Correct ✅
app.post('/api/users', [
  body('email').isEmail().withMessage('Invalid email')
], handleValidationErrors, (req, res) => {
  // Handler runs after validation
});
```

#### 2. Custom Validators Not Working
```javascript
// Problem: Custom validator not throwing errors properly
// Wrong ❌
body('email').custom(async (email) => {
  const user = await User.findOne({ email });
  return !user; // Returns boolean instead of throwing
});

// Correct ✅
body('email').custom(async (email) => {
  const user = await User.findOne({ email });
  if (user) {
    throw new Error('Email already exists');
  }
  return true; // Must return true for success
});
```

#### 3. Sanitization Not Applied
```javascript
// Problem: Sanitization not working as expected
// Wrong ❌
body('name')
  .trim()
  .escape()
  .isLength({ min: 2 }); // Validation runs on original value

// Correct ✅
body('name')
  .trim()        // Sanitize first
  .escape()      // Then more sanitization
  .isLength({ min: 2 }); // Then validate sanitized value
```

#### 4. Error Messages Not Showing
```javascript
// Problem: Generic error messages
// Wrong ❌
body('email').isEmail();

// Correct ✅
body('email')
  .isEmail()
  .withMessage('Please provide a valid email address');

// Problem: Error messages not properly formatted
// Solution: Use consistent error handling
const formatValidationErrors = (errors) => {
  return errors.array().reduce((acc, error) => {
    acc[error.path] = error.msg;
    return acc;
  }, {});
};
```

#### 5. Async Validation Issues
```javascript
// Problem: Async validators not being awaited properly
// Wrong ❌
const validateAsync = (req, res, next) => {
  validationResult(req); // Doesn't wait for async validators
  next();
};

// Correct ✅
const validateAsync = async (req, res, next) => {
  // Run all validations and wait for completion
  await Promise.all(validations.map(validation => validation.run(req)));
  
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  
  next();
};
```

#### 6. Schema Validation Path Issues
```javascript
// Problem: Nested object validation not working
// Wrong ❌
const schema = {
  user.email: { // Invalid path syntax
    isEmail: true
  }
};

// Correct ✅
const schema = {
  'user.email': { // Proper path syntax with quotes
    isEmail: {
      errorMessage: 'Please provide a valid email'
    }
  }
};
```

### Debug Tips

#### 1. Enable Debug Mode
```javascript
// Add debugging to validation chain
body('email')
  .isEmail()
  .withMessage('Invalid email')
  .custom((value) => {
    console.log('Validating email:', value); // Debug log
    return true;
  });
```

#### 2. Log Validation Results
```javascript
const debugValidation = (req, res, next) => {
  const errors = validationResult(req);
  
  console.log('Validation Results:', {
    hasErrors: !errors.isEmpty(),
    errors: errors.array(),
    body: req.body,
    query: req.query,
    params: req.params
  });
  
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  
  next();
};
```

#### 3. Test Individual Validators
```javascript
// Test custom validators in isolation
const testValidator = async () => {
  try {
    const result = await checkUniqueEmail('test@example.com');
    console.log('Validator passed:', result);
  } catch (error) {
    console.log('Validator failed:', error.message);
  }
};

testValidator();
```

## Resources

### Official Documentation
- [Express-Validator Documentation](https://express-validator.github.io/docs/)
- [Validator.js Documentation](https://github.com/validatorjs/validator.js)
- [Express.js Documentation](https://expressjs.com/)

### Useful Packages
- **express-validator**: Main validation library
- **validator**: Underlying validation functions
- **sanitize-html**: Advanced HTML sanitization
- **joi**: Alternative schema-based validation
- **yup**: Another schema validation library
- **multer**: File upload handling
- **sharp**: Image processing and validation
- **bcryptjs**: Password hashing
- **jsonwebtoken**: JWT token handling

### Learning Resources
- [Express-Validator GitHub Repository](https://github.com/express-validator/express-validator)
- [MDN Web Docs - Form Validation](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation)
- [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)

### Testing Tools
- **Jest**: JavaScript testing framework
- **Supertest**: HTTP assertion library
- **Postman**: API testing tool
- **Insomnia**: REST client for testing APIs

### Security Resources
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)
- [Express.js Security Best Practices](https://expressjs.com/en/advanced/best-practice-security.html)

---

This comprehensive guide covers all aspects of express-validator from basic usage to advanced patterns and integration examples. Use it as a reference for implementing robust validation in your Express.js applications, and remember to always prioritize security and user experience when handling user input.
