# Font-Viewer

A Progressive Web Application for browsing, previewing, and selecting fonts through an intuitive drag-and-drop interface.

## Overview

Font-Viewer is part of the Cyberdeck ecosystem and provides a visual catalog of fonts available in the system. It allows users to browse, preview, and compare fonts, and provides an API for other applications to query and use available fonts.

## Features

- **Font Discovery**: Automatically scan and index fonts from the system
- **Font Preview**: Preview fonts with customizable text, size, and style
- **Drag and Drop**: Intuitive drag-and-drop interface for font selection and comparison
- **Collections**: Create and manage font collections and favorites
- **API**: Expose an API for other applications to query and use available fonts
- **Responsive Design**: Works across different devices and screen sizes
- **Offline Support**: Works offline with previously loaded fonts

## Project Structure

```
Font-Viewer/
├── app/                  # Application code
│   ├── assets/           # Static assets
│   │   ├── icons/        # App icons
│   │   └── styles/       # CSS files
│   ├── src/              # Source code
│   │   ├── components/   # Web components
│   │   ├── services/     # Service modules
│   │   └── utils/        # Utility functions
│   └── index.html        # Entry point
├── cicd/                 # CI/CD configuration
├── develop/              # Development files
│   ├── specs/            # Specifications
│   └── tests/            # Test files
├── dist/                 # Distribution files
└── docs/                 # Documentation
```

## Getting Started

### Prerequisites

- Node.js 14 or higher
- Modern web browser (Chrome, Firefox, Safari)

### Installation

1. Clone the repository:
   ```
   git clone https://github.com/username/Font-Viewer.git
   cd Font-Viewer
   ```

2. Install dependencies:
   ```
   npm install
   ```

3. Start the development server:
   ```
   npm start
   ```

4. Open your browser and navigate to `http://localhost:8080`

## Development

### Building

To build the application for production:

```
npm run build
```

This will create a production-ready build in the `dist` directory.

### Testing

To run tests:

```
npm test
```

## Documentation

For more detailed documentation, see the following:

- [Project Specification](./develop/specs/font-viewer-spec.md)
- [Project Plan](./develop/specs/project-plan.md)
- [API Documentation](./docs/api.md)
- [User Guide](./docs/user-guide.md)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Font Viewer is part of the Cyberdeck ecosystem
- Built using WebAX standards and technologies
