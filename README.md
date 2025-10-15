# Revendo & DQ Solutions Integration Documentation

This repository contains the technical documentation and a demonstration page for integrating Revendo's trade-in system with a partner webshop like DQ Solutions.

## Files and Directories

### `/docs/revendo_integration_v2.md`

This file contains the formal **technical documentation** for the integration. It is intended for developers at the partner company who will be responsible for implementing the iframe and backend API calls.

It includes:
- System architecture overview
- Frontend implementation details (iframe URL, `postMessage` events)
- Backend API specification (endpoint, authentication, request/response formats)
- Security best practices

### `/demo/index.html`

This is a **live, interactive testing tool** designed to help developers understand and debug the `postMessage` communication with the Revendo iframe. It is a standalone HTML file that can be opened in a web browser.

Its purpose is to:
- Provide a working example of how to embed the iframe.
- Display all `postMessage` events sent by the iframe in real-time.
- Allow for testing of features like auto-resizing and reloading the iframe.

This file serves as a development aid and is not meant to be deployed to a production environment. It helps clarify the "landing page" question—it's not a customer-facing page but a tool for developers.
