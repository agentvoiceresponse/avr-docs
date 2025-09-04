---
title: AVR Function Calls
description: 
published: true
date: 2025-09-04T14:45:00.551Z
tags: 
editor: markdown
dateCreated: 2025-09-04T14:35:17.641Z
---

# AVR Function Calls 

AVR (Agent Voice Response) supports function calling capabilities, allowing developers to create custom tools that can be invoked during real-time speech-to-speech conversations. This guide explains how to create, implement, and use custom function calls in the AVR system.

## Architecture

The function calling system in AVR consists of two main directories:

- **`avr_tools/`** - Project-provided tools (core functionality)
- **`tools/`** - User custom tools (extensible functionality)

Tools are automatically loaded at runtime and made available to the OpenAI API for function calling.

## Default Function Calls

AVR comes with two essential built-in function calls that are automatically available in every conversation:

### 1. `avr_transfer` - Call Transfer Tool

**Purpose**: Transfers the current call to a specific extension, commonly used to connect users with human operators.

**Usage**: The AI agent can automatically transfer calls when:
- A user requests to speak with a human operator
- The conversation reaches a point where human intervention is needed
- The AI determines it cannot handle a specific request

**Parameters**:
- `transfer_extension` (required): The extension number to transfer to
- `transfer_context` (optional): The context/department for the transfer
- `transfer_priority` (optional): Priority level for the transfer

**Example AI Usage**: "I understand you'd like to speak with a human operator. Let me transfer you to our customer service team."

### 2. `avr_hangup` - Call Termination Tool

**Purpose**: Forces the virtual agent to end the conversation and hang up the call.

**Usage**: The AI agent can automatically hang up when:
- A task or conversation is completed
- No further assistance is needed
- The user's request has been fulfilled
- Maintenance or booking has been completed

**Parameters**: None required

**Example AI Usage**: "Your appointment has been successfully scheduled. Thank you for calling, and have a great day!"

These default tools ensure that every AVR deployment has the essential call management capabilities without requiring additional development.

## Tool Structure

Each tool must follow a specific structure defined by the following properties:

### Required Properties

```javascript
module.exports = {
  name: "tool_name",           // Unique identifier for the tool
  description: "Tool description", // Human-readable description
  input_schema: {},            // JSON Schema for input validation
  handler: async (uuid, args) => {} // Function to execute
};
```

### Property Details

#### `name` (string, required)
- Must be unique across all tools
- Used by OpenAI to identify which tool to call
- Should be descriptive and follow snake_case convention
- Example: `"get_weather"`, `"avr_transfer"`, `"avr_hangup"`

#### `description` (string, required)
- Clear explanation of what the tool does
- Used by OpenAI to understand when to call this tool
- Should be concise but informative
- Example: `"Retrieves weather information for a specific location"`

#### `input_schema` (object, required)
- JSON Schema that defines the expected input parameters
- OpenAI uses this to validate and structure the input data
- Must follow [JSON Schema specification](https://json-schema.org/)

#### `handler` (function, required)
- Async function that executes the tool's logic
- Receives two parameters: `uuid` and `args`
- Must return a string response that will be sent back to the user

## Creating a Custom Tool

### Step 1: Create the Tool File

Create a new JavaScript file in the `tools/` directory:

```bash
touch path/to/tools/my_custom_tool.js
```

### Step 2: Implement the Tool

```javascript
// tools/my_custom_tool.js

module.exports = {
  name: "my_custom_tool",
  description: "Performs a custom operation based on user input",
  
  input_schema: {
    type: "object",
    properties: {
      parameter1: {
        type: "string",
        description: "First parameter description"
      },
      parameter2: {
        type: "number",
        description: "Second parameter description"
      }
    },
    required: ["parameter1"] // parameter2 is optional
  },
  
  handler: async (uuid, { parameter1, parameter2 }) => {
    try {
      // Your custom logic here
      const result = await performCustomOperation(parameter1, parameter2);
      
      return `Operation completed successfully: ${result}`;
    } catch (error) {
      return `Error occurred: ${error.message}`;
    }
  }
};
```

### Step 3: Restart the Services

After adding a new tool, restart the AVR services for changes to take effect.

## Input Schema Examples

### Simple String Parameter

```javascript
input_schema: {
  type: "object",
  properties: {
    location: {
      type: "string",
      description: "The name of the location"
    }
  },
  required: ["location"]
}
```

### Multiple Parameters with Types

```javascript
input_schema: {
  type: "object",
  properties: {
    name: {
      type: "string",
      description: "User's full name"
    },
    age: {
      type: "number",
      description: "User's age in years"
    },
    isActive: {
      type: "boolean",
      description: "Whether the user account is active"
    }
  },
  required: ["name", "age"]
}
```

### Array Parameters

```javascript
input_schema: {
  type: "object",
  properties: {
    items: {
      type: "array",
      items: {
        type: "string"
      },
      description: "List of items to process"
    }
  },
  required: ["items"]
}
```

## Handler Function

### Signature

```javascript
handler: async (uuid, args) => {
  // uuid: Session identifier for tracking
  // args: Object containing the validated input parameters
}
```

### Parameters

- **`uuid`** (string): Unique session identifier for the current conversation
  - Use this for logging, tracking, or making session-specific API calls
  - Example: `"550e8400-e29b-41d4-a716-446655440000"`

- **`args`** (object): Destructured parameters from the input schema
  - Contains the validated parameters as defined in `input_schema`
  - Example: `{ location: "New York" }`

### Return Value

The handler must return a string that will be:
1. Sent back to OpenAI as instructions
2. Converted to speech and sent to the user
3. Used to continue the conversation

### Error Handling

Always implement proper error handling in your handlers:

```javascript
handler: async (uuid, args) => {
  try {
    // Your logic here
    const result = await someAsyncOperation(args);
    return `Success: ${result}`;
  } catch (error) {
    console.error(`Error in ${this.name}:`, error);
    return `I encountered an error: ${error.message}`;
  }
}
```

## Real-World Examples

### Weather Tool

```javascript
// tools/get_weather.js
const axios = require('axios');

module.exports = {
  name: "get_weather",
  description: "Retrieves current weather information for a specified location",
  
  input_schema: {
    type: "object",
    properties: {
      location: {
        type: "string",
        description: "The city or location name"
      },
      units: {
        type: "string",
        enum: ["celsius", "fahrenheit"],
        description: "Temperature units (default: celsius)"
      }
    },
    required: ["location"]
  },
  
  handler: async (uuid, { location, units = "celsius" }) => {
    try {
      // Simulate weather API call
      const weather = await getWeatherData(location);
      
      return `The weather in ${location} is currently ${weather.condition} with a temperature of ${weather.temperature}°${units === "fahrenheit" ? "F" : "C"}.`;
    } catch (error) {
      return `I'm sorry, I couldn't retrieve the weather for ${location}. Please try again.`;
    }
  }
};
```

### Call Transfer Tool

```javascript
// tools/avr_transfer.js
const axios = require('axios');

module.exports = {
  name: "avr_transfer",
  description: "Transfers the current call to a specific extension or department",
  
  input_schema: {
    type: "object",
    properties: {
      extension: {
        type: "string",
        description: "The extension number to transfer to"
      },
      department: {
        type: "string",
        description: "The department name (optional)"
      }
    },
    required: ["extension"]
  },
  
  handler: async (uuid, { extension, department }) => {
    try {
      const response = await axios.post(`${process.env.AMI_URL}/transfer`, {
        uuid,
        exten: extension,
        context: department || "default"
      });
      
      return `I'm transferring your call to extension ${extension}${department ? ` in the ${department} department` : ''}. Please hold while I connect you.`;
    } catch (error) {
      return `I'm sorry, I couldn't complete the transfer. Please try again or contact support.`;
    }
  }
};
```

## Best Practices

### 1. Naming Conventions
- Use descriptive, lowercase names with underscores
- Prefix AVR-specific tools with `avr_`
- Keep names short but meaningful

### 2. Error Handling
- Always wrap your logic in try-catch blocks
- Return user-friendly error messages
- Log errors for debugging purposes

### 3. Input Validation
- Use comprehensive JSON schemas
- Make required fields explicit
- Provide clear descriptions for all parameters

### 4. Response Format
- Keep responses conversational and natural
- Include relevant information from the operation
- Use appropriate language for the context

### 5. Performance
- Keep handlers lightweight and fast
- Use async/await for external API calls
- Avoid blocking operations

## Troubleshooting

### Common Issues

1. **Tool not loading**
   - Check file syntax and exports
   - Verify file is in the correct directory
   - Restart the service after changes

2. **Handler not executing**
   - Check logs for function call attempts
   - Verify the tool name matches exactly
   - Ensure the handler function is properly exported

3. **Parameter validation errors**
   - Review your JSON schema syntax
   - Check required field definitions
   - Verify parameter types match expected values

### Debug Tips

- Add console.log statements in your handlers
- Check the AVR service logs for errors
- Verify tool loading in the startup logs
- Test with simple tools first before complex implementations

## Environment Variables

Your tools can access environment variables through `process.env`:

```javascript
handler: async (uuid, args) => {
  const apiKey = process.env.MY_API_KEY;
  const baseUrl = process.env.MY_SERVICE_URL;
  
  // Use these in your tool logic
}
```

## Security Considerations

- Never expose sensitive information in tool responses
- Validate all input parameters thoroughly
- Use environment variables for API keys and secrets
- Implement rate limiting for external API calls
- Sanitize user input to prevent injection attacks

## Conclusion

Function calls in AVR provide a powerful way to extend the system's capabilities. By following this guide, you can create robust, reliable tools that enhance the user experience and provide valuable functionality during voice conversations.

The built-in `avr_transfer` and `avr_hangup` tools ensure that every deployment has essential call management capabilities, while the extensible `tools/` directory allows you to add custom functionality specific to your use case.

For additional support and examples, refer to the existing tools in the `avr_tools/` directory and the main AVR documentation.