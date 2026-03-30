# OpenWISP WiFi Login Pages Demo

A modernization project for the OpenWISP WiFi login pages, focused on architectural refactoring, improved performance, and modern web standards.

## 🚀 Overview
This repository serves as a demonstration for the **WiFi Login Pages Modernization** project. The goal is to evolve the existing platform from a monolithic structure to a modular, high-performance architecture using modern React patterns.

## ✨ Key Features
- **Modular Component Architecture**: Refactored status components (Auth, Verification, Payment, Session, Captive Portal) into focused, testable layers.
- **Unified Responsive Header**: Single-structure header replacing duplicated desktop/mobile blocks with responsive CSS.
- **RFC 8908 Support**: Native Captive Portal API integration for faster, more reliable status detection on modern devices.
- **React 19 & RTL**: Upgraded ecosystem with robust testing using React Testing Library.
- **Clean Redirect Logic**: Decentralized routing logic to eliminate race conditions and redundant HTTP requests.

## 🛠 Tech Stack
- **Framework**: React 19
- **Testing**: React Testing Library (RTL)
- **Styling**: Vanilla CSS (Flexbox/Grid)
- **Protocols**: RFC 8908 (Captive Portal API)

## 🌐 Demo
Check out the live prototype here:
**[View Demo](https://openwisp-wifi-pages-demo.vercel.app)**

## 📂 Repository Contents
- `index.html`: A static demonstration of the modernized WiFi login interface.
- `README.md`: Project overview and documentation.

---
*Created as part of the Google Summer of Code 2026 proposal for OpenWISP.*
