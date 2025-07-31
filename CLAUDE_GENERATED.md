# Claude Generated Code

**Task**: Objective: Audit the entire codebase and identify improvements in:  🔐 Security Detect insecure patterns (e.g. unvalidated inputs, unsafe data handling).  Suggest best practices (e.g. input sanitization, environment variable usage).  🛠️ Maintainability Flag complex or duplicate code, unclear logic, poor naming.  Recommend modularization and better structure.  🚀 Performance Spot inefficient loops, redundant logic, unnecessary API calls.  Suggest caching or optimization where needed.  🧹 Clean-up Find dead code, unused variables/functions/imports.  List outdated or unnecessary dependencies.

**Generated on**: Thu Jul 31 05:55:54 UTC 2025

## Generated Content

```
Based on my audit of the Ionic React TypeScript government billing/invoicing application codebase, here are the key improvements needed across security, maintainability, performance, and clean-up areas:

## FILES_TO_CREATE

### 1. Environment Configuration
**File**: `src/config/env.ts`

### 2. Input Validation Utilities  
**File**: `src/utils/validation.ts`

### 3. Security Utilities
**File**: `src/utils/security.ts`

### 4. Error Handling System
**File**: `src/utils/errorHandler.ts`

### 5. Constants File
**File**: `src/constants/app.ts`

### 6. TypeScript Types
**File**: `src/types/index.ts`

### 7. Storage Service
**File**: `src/services/StorageService.ts`

### 8. Web3Provider Improvements
**File**: `src/utils/Web3Provider.tsx`

## FILES_TO_MODIFY

1. `package.json` - Update dependencies and remove unused ones
2. `src/App/App.js` - Convert to TypeScript and improve structure
3. `src/socialcalc/AppGeneral.js` - Security and validation improvements
4. `src/Menu/Menu.js` - Convert to TypeScript (assumed to exist)
5. `src/Files/Files.js` - Convert to TypeScript (assumed to exist)
6. `src/storage/LocalStorage.js` - Fix TypeScript issues and improve security

## CODE_CHANGES

### FILES_TO_CREATE

#### `src/config/env.ts`
```typescript
// Environment configuration with validation
export interface AppConfig {
  NODE_ENV: string;
  API_BASE_URL: string;
  WEB3_NETWORK: string;
  APP_VERSION: string;
  MAX_FILE_SIZE: number;
  ENCRYPTION_ENABLED: boolean;
}

const getEnvVar = (key: string, defaultValue?: string): string => {
  const value = process.env[`REACT_APP_${key}`] || defaultValue;
  if (!value) {
    throw new Error(`Environment variable REACT_APP_${key} is required`);
  }
  return value;
};

export const config: AppConfig = {
  NODE_ENV: getEnvVar('NODE_ENV', 'development'),
  API_BASE_URL: getEnvVar('API_BASE_URL', ''),
  WEB3_NETWORK: getEnvVar('WEB3_NETWORK', 'mainnet'),
  APP_VERSION: getEnvVar('APP_VERSION', '1.0.0'),
  MAX_FILE_SIZE: parseInt(getEnvVar('MAX_FILE_SIZE', '5242880')), // 5MB
  ENCRYPTION_ENABLED: getEnvVar('ENCRYPTION_ENABLED', 'false') === 'true'
};

// Validate config on load
if (config.NODE_ENV === 'production' && !config.API_BASE_URL) {
  throw new Error('API_BASE_URL is required in production');
}
```

#### `src/utils/validation.ts`
```typescript
import DOMPurify from 'dompurify';

export interface ValidationResult {
  isValid: boolean;
  errors: string[];
}

export class ValidationUtils {
  
  static sanitizeInput(input: string): string {
    if (typeof input !== 'string') return '';
    return DOMPurify.sanitize(input.trim());
  }

  static validateEmail(email: string): ValidationResult {
    const sanitized = this.sanitizeInput(email);
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    
    return {
      isValid: emailRegex.test(sanitized),
      errors: emailRegex.test(sanitized) ? [] : ['Invalid email format']
    };
  }

  static validatePhone(phone: string): ValidationResult {
    const sanitized = this.sanitizeInput(phone);
    const phoneRegex = /^[\+]?[1-9][\d]{0,15}$/;
    
    return {
      isValid: phoneRegex.test(sanitized.replace(/[\s\-\(\)]/g, '')),
      errors: phoneRegex.test(sanitized.replace(/[\s\-\(\)]/g, '')) ? [] : ['Invalid phone format']
    };
  }

  static validateCurrency(amount: string): ValidationResult {
    const sanitized = this.sanitizeInput(amount);
    const currencyRegex = /^\d+(\.\d{0,2})?$/;
    const numValue = parseFloat(sanitized);
    
    const errors: string[] = [];
    
    if (!currencyRegex.test(sanitized)) {
      errors.push('Invalid currency format');
    }
    
    if (numValue < 0) {
      errors.push('Amount cannot be negative');
    }
    
    if (numValue > 999999.99) {
      errors.push('Amount exceeds maximum allowed');
    }
    
    return {
      isValid: errors.length === 0,
      errors
    };
  }

  static validateRequired(value: any, fieldName: string): ValidationResult {
    const isEmpty = value === null || value === undefined || 
                   (typeof value === 'string' && value.trim().length === 0);
    
    return {
      isValid: !isEmpty,
      errors: isEmpty ? [`${fieldName} is required`] : []
    };
  }

  static validateMaxLength(value: string, maxLength: number, fieldName: string): ValidationResult {
    const sanitized = this.sanitizeInput(value);
    
    return {
      isValid: sanitized.length <= maxLength,
      errors: sanitized.length <= maxLength ? [] : [`${fieldName} exceeds maximum length of ${maxLength}`]
    };
  }
}
```

#### `src/utils/security.ts`
```typescript
import CryptoJS from 'crypto-js';
import { config } from '../config/env';

export class SecurityUtils {
  
  private static readonly STORAGE_ENCRYPTION_KEY = 'invoice_app_key_2024';
  
  static encryptData(data: string): string {
    if (!config.ENCRYPTION_ENABLED) return data;
    
    try {
      return CryptoJS.AES.encrypt(data, this.STORAGE_ENCRYPTION_KEY).toString();
    } catch (error) {
      console.error('Encryption failed:', error);
      throw new Error('Failed to encrypt data');
    }
  }
  
  static decryptData(encryptedData: string): string {
    if (!config.ENCRYPTION_ENABLED) return encryptedData;
    
    try {
      const bytes = CryptoJS.AES.decrypt(encryptedData, this.STORAGE_ENCRYPTION_KEY);
      return bytes.toString(CryptoJS.enc.Utf8);
    } catch (error) {
      console.error('Decryption failed:', error);
      throw new Error('Failed to decrypt data');
    }
  }
  
  static generateSecureId(): string {
    return CryptoJS.lib.WordArray.random(16).toString();
  }
  
  static hashPassword(password: string): string {
    const salt = CryptoJS.lib.WordArray.random(16);
    const hash = CryptoJS.PBKDF2(password, salt, { keySize: 256/32, iterations: 10000 });
    return salt.toString() + ':' + hash.toString();
  }
  
  static verifyPassword(password: string, hashedPassword: string): boolean {
    try {
      const [salt, hash] = hashedPassword.split(':');
      const computedHash = CryptoJS.PBKDF2(password, CryptoJS.enc.Hex.parse(salt), { 
        keySize: 256/32, 
        iterations: 10000 
      });
      return hash === computedHash.toString();
    } catch {
      return false;
    }
  }
  
  static isSecureContext(): boolean {
    return window.isSecureContext || location.protocol === 'https:';
  }
}
```

#### `src/utils/errorHandler.ts`
```typescript
export enum ErrorType {
  VALIDATION = 'VALIDATION',
  STORAGE = 'STORAGE',
  NETWORK = 'NETWORK',
  SECURITY = 'SECURITY',
  UNKNOWN = 'UNKNOWN'
}

export interface AppError {
  type: ErrorType;
  message: string;
  code?: string;
  details?: any;
  timestamp: Date;
}

export class ErrorHandler {
  
  static createError(type: ErrorType, message: string, code?: string, details?: any): AppError {
    return {
      type,
      message,
      code,
      details,
      timestamp: new Date()
    };
  }
  
  static handleError(error: any): AppError {
    console.error('Error occurred:', error);
    
    if (error instanceof AppError) {
      return error;
    }
    
    // Network errors
    if (error.name === 'NetworkError' || error.code === 'NETWORK_ERROR') {
      return this.createError(ErrorType.NETWORK, 'Network connection failed', error.code, error);
    }
    
    // Storage errors
    if (error.name === 'QuotaExceededError' || error.code === 22) {
      return this.createError(ErrorType.STORAGE, 'Storage quota exceeded', 'QUOTA_EXCEEDED', error);
    }
    
    // Security errors
    if (error.name === 'SecurityError') {
      return this.createError(ErrorType.SECURITY, 'Security violation', 'SECURITY_ERROR', error);
    }
    
    // Default unknown error
    return this.createError(
      ErrorType.UNKNOWN, 
      error.message || 'An unexpected error occurred',
      error.code,
      error
    );
  }
  
  static logError(error: AppError): void {
    const logData = {
      timestamp: error.timestamp.toISOString(),
      type: error.type,
      message: error.message,
      code: error.code,
      userAgent: navigator.userAgent,
      url: window.location.href
    };
    
    // In production, send to error reporting service
    if (process.env.NODE_ENV === 'production') {
      // Send to error reporting service (e.g., Sentry, LogRocket)
      console.error('Production Error:', logData);
    } else {
      console.error('Development Error:', logData, error.details);
    }
  }
}
```

#### `src/constants/app.ts`
```typescript
export const APP_CONSTANTS = {
  APP_NAME: 'Invoice Suite',
  VERSION: '1.0.0',
  STORAGE_PREFIX: 'invoice_app_',
  
  FILE_CONSTRAINTS: {
    MAX_SIZE: 5 * 1024 * 1024, // 5MB
    ALLOWED_TYPES: ['application/json', 'text/plain'],
    MAX_NAME_LENGTH: 255
  },
  
  VALIDATION_LIMITS: {
    MAX_DESCRIPTION_LENGTH: 1000,
    MAX_NAME_LENGTH: 100,
    MAX_ADDRESS_LENGTH: 200,
    MAX_PHONE_LENGTH: 20,
    MAX_EMAIL_LENGTH: 254,
    MIN_PASSWORD_LENGTH: 8
  },
  
  DEVICE_TYPES: {
    DEFAULT: 'default',
    IPAD: 'iPad',
    IPHONE: 'iPhone',
    IPOD: 'iPod'
  },
  
  SHEETS: {
    TYPE_I: 'typei',
    TYPE_II: 'typeii', 
    TYPE_III: 'typeiii',
    TYPE_IV: 'typeiv'
  }
} as const;

export type DeviceType = typeof APP_CONSTANTS.DEVICE_TYPES[keyof typeof APP_CONSTANTS.DEVICE_TYPES];
export type SheetType = typeof APP_CONSTANTS.SHEETS[keyof typeof APP_CONSTANTS.SHEETS];
```

#### `src/types/index.ts`
```typescript
export interface InvoiceData {
  id: string;
  name: string;
  created: string;
  modified: string;
  content: string;
  password?: string;
  encrypted?: boolean;
}

export interface AppState {
  selectedFile: string;
  device: DeviceType;
  listFiles: boolean;
  loading: boolean;
  error?: AppError;
}

export interface SheetFooter {
  name: string;
  index: number;
  isActive: boolean;
}

export interface WorkbookData {
  msc: any; // SocialCalc workbook data structure
  footers: SheetFooter[];
}

export interface FileOperationResult {
  success: boolean;
  data?: any;
  error?: string;
}

import { DeviceType } from '../constants/app';
import { AppError } from '../utils/errorHandler';
```

#### `src/services/StorageService.ts`
```typescript
import { InvoiceData, FileOperationResult } from '../types';
import { SecurityUtils } from '../utils/security';
import { ValidationUtils } from '../utils/validation';
import { ErrorHandler, ErrorType } from '../utils/errorHandler';
import { APP_CONSTANTS } from '../constants/app';

export class StorageService {
  private readonly storage: Storage;
  private readonly PREFIX = APP_CONSTANTS.STORAGE_PREFIX;

  constructor() {
    this.storage = window.localStorage;
    this.validateStorageAvailability();
  }

  private validateStorageAvailability(): void {
    try {
      const test = '__storage_test__';
      this.storage.setItem(test, test);
      this.storage.removeItem(test);
    } catch (error) {
      throw ErrorHandler.createError(
        ErrorType.STORAGE,
        'Local storage is not available',
        'STORAGE_UNAVAILABLE'
      );
    }
  }

  private getStorageKey(filename: string): string {
    const sanitized = ValidationUtils.sanitizeInput(filename);
    return `${this.PREFIX}${sanitized}`;
  }

  async saveFile(file: InvoiceData): Promise<FileOperationResult> {
    try {
      // Validate input
      const nameValidation = ValidationUtils.validateRequired(file.name, 'File name');
      if (!nameValidation.isValid) {
        return { success: false, error: nameValidation.errors.join(', ') };
      }

      const maxLengthValidation = ValidationUtils.validateMaxLength(
        file.name, 
        APP_CONSTANTS.FILE_CONSTRAINTS.MAX_NAME_LENGTH, 
        'File name'
      );
      if (!maxLengthValidation.isValid) {
        return { success: false, error: maxLengthValidation.errors.join(', ') };
      }

      // Prepare data for storage
      const dataToStore = {
        id: file.id || SecurityUtils.generateSecureId(),
        created: file.created || new Date().toISOString(),
        modified: new Date().toISOString(),
        content: file.password ? SecurityUtils.encryptData(file.content) : file.content,
        name: ValidationUtils.sanitizeInput(file.name),
        encrypted: !!file.password
      };

      // Check storage quota
      const serializedData = JSON.stringify(dataToStore);
      if (serializedData.length > APP_CONSTANTS.FILE_CONSTRAINTS.MAX_SIZE) {
        return { 
          success: false, 
          error: `File size exceeds limit of ${APP_CONSTANTS.FILE_CONSTRAINTS.MAX_SIZE / 1024 / 1024}MB` 
        };
      }

      const storageKey = this.getStorageKey(file.name);
      this.storage.setItem(storageKey, serializedData);

      return { success: true, data: dataToStore };

    } catch (error) {
      const appError = ErrorHandler.handleError(error);
      ErrorHandler.logError(appError);
      
      return { 
        success: false, 
        error: appError.type === ErrorType.STORAGE ? 
          'Storage quota exceeded or unavailable' : 
          'Failed to save file' 
      };
    }
  }

  async loadFile(filename: string): Promise<FileOperationResult> {
    try {
      const storageKey = this.getStorageKey(filename);
      const data = this.storage.getItem(storageKey);
      
      if (!data) {
        return { success: false, error: 'File not found' };
      }

      const parsedData = JSON.parse(data);
      
      // Decrypt if necessary
      if (parsedData.encrypted && parsedData.content) {
        try {
          parsedData.content = SecurityUtils.decryptData(parsedData.content);
        } catch (error) {
          return { success: false, error: 'Failed to decrypt file - invalid password or corrupted data' };
        }
      }

      return { success: true, data: parsedData };

    } catch (error) {
      const appError = ErrorHandler.handleError(error);
      ErrorHandler.logError(appError);
      
      return { success: false, error: 'Failed to load file' };
    }
  }

  async deleteFile(filename: string): Promise<FileOperationResult> {
    try {
      const storageKey = this.getStorageKey(filename);
      
      if (!this.storage.getItem(storageKey)) {
        return { success: false, error: 'File not found' };
      }

      this.storage.removeItem(storageKey);
      return { success: true };

    } catch (error) {
      const appError = ErrorHandler.handleError(error);
      ErrorHandler.logError(appError);
      
      return { success: false, error: 'Failed to delete file' };
    }
  }

  async listFiles(): Promise<FileOperationResult> {
    try {
      const files: InvoiceData[] = [];
      
      for (let i = 0; i < this.storage.length; i++) {
        const key = this.storage.key(i);
        if (key?.startsWith(this.PREFIX)) {
          const data = this.storage.getItem(key);
          if (data) {
            try {
              const parsedData = JSON.parse(data);
              // Don't include content in list view for performance
              files.push({
                id: parsedData.id,
                name: parsedData.name,
                created: parsedData.created,
                modified: parsedData.modified,
                content: '', // Empty for list view
                encrypted: parsedData.encrypted
              });
            } catch {
              // Skip corrupted files
              continue;
            }
          }
        }
      }

      return { success: true, data: files.sort((a, b) => 
        new Date(b.modified).getTime() - new Date(a.modified).getTime()) 
      };

    } catch (error) {
      const appError = ErrorHandler.handleError(error);
      ErrorHandler.logError(appError);
      
      return { success: false, error: 'Failed to list files' };
    }
  }

  getStorageInfo(): { used: number; available: number; percentage: number } {
    let used = 0;
    
    try {
      for (let i = 0; i < this.storage.length; i++) {
        const key = this.storage.key(i);
        if (key?.startsWith(this.PREFIX)) {
          const data = this.storage.getItem(key);
          if (data) used += data.length;
        }
      }
    } catch {
      // If we can't calculate, return safe defaults
      return { used: 0, available: 5242880, percentage: 0 };
    }

    const available = APP_CONSTANTS.FILE_CONSTRAINTS.MAX_SIZE;
    const percentage = (used / available) * 100;
    
    return { used, available, percentage };
  }
}
```

#### `src/utils/Web3Provider.tsx`
```typescript
import React, { createContext, useContext, useEffect, useState, ReactNode } from 'react';
import { WagmiConfig, createConfig, configureChains } from 'wagmi';
import { mainnet, polygon, optimism, arbitrum } from 'wagmi/chains';
import { publicProvider } from 'wagmi/providers/public';
import { ConnectKitProvider, getDefaultConfig } from 'connectkit';
import { ErrorHandler, ErrorType } from './errorHandler';
import { config as appConfig } from '../config/env';

interface Web3ContextType {
  isConnected: boolean;
  account?: string;
  chainId?: number;
  error?: string;
}

const Web3Context = createContext<Web3ContextType>({
  isConnected: false
});

export const useWeb3 = () => {
  const context = useContext(Web3Context);
  if (!context) {
    throw new Error('useWeb3 must be used within a Web3Provider');
  }
  return context;
};

interface Web3ProviderProps {
  children: ReactNode;
}

export const Web3Provider: React.FC<Web3ProviderProps> = ({ children }) => {
  const [web3State, setWeb3State] = useState<Web3ContextType>({
    isConnected: false
  });

  // Configure chains based on environment
  const chains = [mainnet, polygon, optimism, arbitrum];
  
  const { publicClient, webSocketPublicClient } = configureChains(chains, [
    publicProvider()
  ]);

  const config = createConfig(
    getDefaultConfig({
      appName: appConfig.NODE_ENV === 'production' ? 'Invoice Suite' : 'Invoice Suite (Dev)',
      chains,
      publicClient,
      webSocketPublicClient,
      walletConnectProjectId: process.env.REACT_APP_WALLETCONNECT_PROJECT_ID || '',
    })
  );

  useEffect(() => {
    // Security check for Web3 context
    if (!window.isSecureContext && appConfig.NODE_ENV === 'production') {
      const error = ErrorHandler.createError(
        ErrorType.SECURITY,
        'Web3 features require a secure context (HTTPS)',
        'INSECURE_CONTEXT'
      );
      ErrorHandler.logError(error);
      setWeb3State(prev => ({ ...prev, error: error.message }));
    }
  }, []);

  return (
    <WagmiConfig config={config}>
      <ConnectKitProvider
        theme="auto"
        mode="light"
        options={{
          initialChainId: 0,
          enforceSupportedChains: true,
          disclaimer: (
            <div style={{ padding: '1rem', fontSize: '0.875rem' }}>
              <p>
                This application handles sensitive financial data. Only connect wallets you trust 
                and verify all transactions before signing.
              </p>
            </div>
          )
        }}
      >
        <Web3Context.Provider value={web3State}>
          {children}
        </Web3Context.Provider>
      </ConnectKitProvider>
    </WagmiConfig>
  );
};
```

### FILES_TO_MODIFY

#### `package.json` (Updated dependencies and cleanup)
```json
{
  "name": "invoice-suite",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "@ionic/react": "^7.5.0",
    "@ionic/react-router": "^7.5.0",
    "@tanstack/react-query": "^5.49.2",
    "@types/react": "^18.2.42",
    "@types/react-dom": "^18.2.17",
    "connectkit": "^1.8.2",
    "crypto-js": "^4.2.0",
    "dompurify": "^3.0.5",
    "ethers": "^6.8.1",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-router": "^5.3.4",
    "react-router-dom": "^5.3.4",
    "typescript": "^5.2.2",
    "viem": "^2.0.0",
    "wagmi": "^2.10.9"
  },
  "devDependencies": {
    "@ionic/cli": "^7.2.0",
    "@testing-library/jest-dom": "^5.17.0",
    "@testing-library/react": "^13.4.0",
    "@testing-library/user-event": "^13.5.0",
    "@types/crypto-js": "^4.2.1",
    "@types/dompurify": "^3.0.5",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject",
    "ionic:build": "npm run build",
    "ionic:serve": "npm start"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

#### `src/App/App.tsx` (Converted from JS to TypeScript with improvements)
```typescript
import React, { Component } from "react";
import { IonApp, IonContent, IonPage, IonLoading, IonAlert } from "@ionic/react";
import { ConnectKitButton } from "connectkit";
import "./App.css";
import * as AppGeneral from "../socialcalc/AppGeneral";
import { DATA } from "../app-data";
import Menu from "../Menu/Menu";
import Files from "../Files/Files";
import { StorageService } from "../services/StorageService";
import { ValidationUtils } from "../utils/validation";
import { ErrorHandler, ErrorType } from "../utils/errorHandler";
import { AppState, InvoiceData } from "../types";
import { APP_CONSTANTS, DeviceType } from "../constants/app";

interface Props {}

class App extends Component<Props, AppState> {
  private storageService: StorageService;

  constructor(props: Props) {
    super(props);
    
    this.storageService = new StorageService();
    
    this.state = {
      selectedFile: "default",
      device: AppGeneral.getDeviceType() as DeviceType,
      listFiles: false,
      loading: false,
    };

    this.updateSelectedFile = this.updateSelectedFile.bind(this);
    this.toggleListFiles = this.toggleListFiles.bind(this);
    this.handleError = this.handleError.bind(this);
  }

  componentDidMount() {
    this.initializeApp();
  }

  private async initializeApp(): Promise<void> {
    try {
      this.setState({ loading: true });
      
      // Security check
      if (!window.isSecureContext && process.env.NODE_ENV === 'production') {
        throw ErrorHandler.createError(
          ErrorType.SECURITY,
          'Application requires HTTPS in production',
          'INSECURE_CONTEXT'
        );
      }

      // Initialize spreadsheet with validation
      const deviceData = this.getDeviceData();
      if (deviceData) {
        AppGeneral.initializeApp(deviceData);
      }
      
    } catch (error) {
      this.handleError(error);
    } finally {
      this.setState({ loading: false });
    }
  }

  private getDeviceData() {
    const { device, selectedFile } = this.state;
    
    if (!DATA.ledger || !DATA.ledger[device] || !DATA.ledger[device][selectedFile]) {
      console.warn(`No data found for device: ${device}, file: ${selectedFile}`);
      return DATA.ledger?.default?.default || null;
    }
    
    return DATA.ledger[device][selectedFile];
  }

  private handleError(error: any): void {
    const appError = ErrorHandler.handleError(error);
    ErrorHandler.logError(appError);
    
    this.setState({ 
      error: appError,
      loading: false 
    });
  }

  updateSelectedFile(selectedFile: string): void {
    // Validate input
    const sanitizedFile = ValidationUtils.sanitizeInput(selectedFile);
    if (!sanitizedFile) {
      this.handleError(ErrorHandler.createError(
        ErrorType.VALIDATION,
        'Invalid file selection',
        'INVALID_FILE'
      ));
      return;
    }

    this.setState({ 
      selectedFile: sanitizedFile,
      listFiles: false 
    }, () => {
      // Re-initialize with new file
      const deviceData = this.getDeviceData();
      if (deviceData) {
        try {
          AppGeneral.initializeApp(deviceData);
        } catch (error) {
          this.handleError(error);
        }
      }
    });
  }

  toggleListFiles(): void {
    this.setState((prevState) => ({
      listFiles: !prevState.listFiles,
    }));
  }

  private clearError = (): void => {
    this.setState({ error: undefined });
  };

  render() {
    const { selectedFile, device, listFiles, loading, error } = this.state;

    return (
      <IonApp>
        <IonPage>
          {loading && (
            <IonLoading
              isOpen={loading}
              message="Initializing application..."
            />
          )}
          
          {error && (
            <IonAlert
              isOpen={!!error}
              header="Error"
              message={error.message}
              buttons={[
                {
                  text: 'OK',
                  handler: this.clearError
                }
              ]}
            />
          )}
          
          <IonContent>
            <div className="App">
              <header className="App-header">
                <div className="app-title">
                  <h1>{APP_CONSTANTS.APP_NAME}</h1>
                  <span className="version">v{APP_CONSTANTS.VERSION}</span>
                </div>
                
                <div className="connection-status">
                  <ConnectKitButton />
                </div>
              </header>

              <div className="main-content">
                {!listFiles && (
                  <Menu
                    updateSelectedFile={this.updateSelectedFile}
                    toggleListFiles={this.toggleListFiles}
                    selectedFile={selectedFile}
                    device={device}
                  />
                )}

                {listFiles && (
                  <Files
                    toggleListFiles={this.toggleListFiles}
                    updateSelectedFile={this.updateSelectedFile}
                    storageService={this.storageService}
                  />
                )}

                {!listFiles && (
                  <div className="spreadsheet-container">
                    <div id="tableeditor" />
                    <div id="workbookControl" />
                  </div>
                )}
              </div>

              <footer className="app-footer">
                <div className="device-info">
                  Device: {device} | File: {selectedFile}
                
```
