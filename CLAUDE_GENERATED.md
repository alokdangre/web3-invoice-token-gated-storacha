# Claude Generated Code

**Task**: Objective: Help design and implement new features based on the current roadmap or project needs. Focus on clean, scalable, and testable code  Claude Tasks:  🧩 Feature Implementation Read the existing codebase and integrate new feature modules where appropriate.  Ensure new code follows existing patterns and conventions.  🗂️ Example Features (replace/add specifics as needed): Add user role-based access control  Implement export to PDF  Add multi-language support  Enable dark mode toggle  🧪 Testing & Documentation Add unit/integration tests for new functionality.  Update README or in-code comments if required.

**Generated on**: Thu Jul 31 06:02:27 UTC 2025

## Generated Content

```
Looking at the codebase, I'll implement user role-based access control, export to PDF functionality, multi-language support, and a dark mode toggle. Based on the existing structure, I'll create a comprehensive solution that integrates well with the current Ionic React TypeScript application.

## FILES_TO_CREATE

1. `src/auth/AuthContext.tsx`
2. `src/auth/types.ts`
3. `src/auth/RoleGuard.tsx`
4. `src/utils/pdfExport.ts`
5. `src/i18n/index.ts`
6. `src/i18n/translations/en.ts`
7. `src/i18n/translations/es.ts`
8. `src/i18n/translations/fr.ts`
9. `src/theme/ThemeContext.tsx`
10. `src/theme/theme.css`
11. `src/components/Settings/Settings.tsx`
12. `src/components/ExportButton/ExportButton.tsx`
13. `src/components/LanguageSelector/LanguageSelector.tsx`
14. `src/components/ThemeToggle/ThemeToggle.tsx`
15. `src/hooks/useAuth.ts`
16. `src/hooks/useTheme.ts`
17. `src/hooks/useTranslation.ts`

## FILES_TO_MODIFY

1. `src/index.js`
2. `src/App/App.js`
3. `src/Menu/Menu.js`
4. `package.json`

## CODE_CHANGES

### New Files

**src/auth/types.ts**
```typescript
export interface User {
  id: string;
  email: string;
  name: string;
  role: UserRole;
  permissions: Permission[];
}

export enum UserRole {
  ADMIN = 'admin',
  MANAGER = 'manager',
  USER = 'user',
  VIEWER = 'viewer'
}

export enum Permission {
  CREATE_INVOICE = 'create_invoice',
  EDIT_INVOICE = 'edit_invoice',
  DELETE_INVOICE = 'delete_invoice',
  VIEW_INVOICE = 'view_invoice',
  EXPORT_PDF = 'export_pdf',
  MANAGE_USERS = 'manage_users',
  VIEW_SETTINGS = 'view_settings'
}

export interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

export interface AuthContextType extends AuthState {
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  hasPermission: (permission: Permission) => boolean;
  hasRole: (role: UserRole) => boolean;
}
```

**src/auth/AuthContext.tsx**
```typescript
import React, { createContext, useContext, useReducer, useEffect } from 'react';
import { User, UserRole, Permission, AuthState, AuthContextType } from './types';

const initialState: AuthState = {
  user: null,
  isAuthenticated: false,
  isLoading: true,
  error: null,
};

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_ERROR'; payload: string }
  | { type: 'LOGOUT' }
  | { type: 'INIT_COMPLETE' };

const authReducer = (state: AuthState, action: AuthAction): AuthState => {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, isLoading: true, error: null };
    case 'LOGIN_SUCCESS':
      return {
        ...state,
        user: action.payload,
        isAuthenticated: true,
        isLoading: false,
        error: null,
      };
    case 'LOGIN_ERROR':
      return {
        ...state,
        user: null,
        isAuthenticated: false,
        isLoading: false,
        error: action.payload,
      };
    case 'LOGOUT':
      return {
        ...state,
        user: null,
        isAuthenticated: false,
        error: null,
      };
    case 'INIT_COMPLETE':
      return { ...state, isLoading: false };
    default:
      return state;
  }
};

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialState);

  useEffect(() => {
    // Check for stored auth data on initialization
    const storedUser = localStorage.getItem('user');
    if (storedUser) {
      try {
        const user = JSON.parse(storedUser);
        dispatch({ type: 'LOGIN_SUCCESS', payload: user });
      } catch (error) {
        localStorage.removeItem('user');
      }
    }
    dispatch({ type: 'INIT_COMPLETE' });
  }, []);

  const login = async (email: string, password: string) => {
    dispatch({ type: 'LOGIN_START' });
    
    try {
      // Simulate API call - replace with actual authentication
      await new Promise(resolve => setTimeout(resolve, 1000));
      
      // Mock user data - replace with actual API response
      const mockUser: User = {
        id: '1',
        email,
        name: email.split('@')[0],
        role: email.includes('admin') ? UserRole.ADMIN : UserRole.USER,
        permissions: email.includes('admin') 
          ? Object.values(Permission)
          : [Permission.VIEW_INVOICE, Permission.CREATE_INVOICE, Permission.EDIT_INVOICE, Permission.EXPORT_PDF]
      };

      localStorage.setItem('user', JSON.stringify(mockUser));
      dispatch({ type: 'LOGIN_SUCCESS', payload: mockUser });
    } catch (error) {
      dispatch({ type: 'LOGIN_ERROR', payload: 'Invalid credentials' });
    }
  };

  const logout = () => {
    localStorage.removeItem('user');
    dispatch({ type: 'LOGOUT' });
  };

  const hasPermission = (permission: Permission): boolean => {
    return state.user?.permissions.includes(permission) || false;
  };

  const hasRole = (role: UserRole): boolean => {
    return state.user?.role === role || false;
  };

  const value: AuthContextType = {
    ...state,
    login,
    logout,
    hasPermission,
    hasRole,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

**src/auth/RoleGuard.tsx**
```typescript
import React from 'react';
import { IonAlert } from '@ionic/react';
import { useAuth } from './AuthContext';
import { Permission, UserRole } from './types';

interface RoleGuardProps {
  children: React.ReactNode;
  permission?: Permission;
  role?: UserRole;
  fallback?: React.ReactNode;
}

export const RoleGuard: React.FC<RoleGuardProps> = ({ 
  children, 
  permission, 
  role, 
  fallback 
}) => {
  const { hasPermission, hasRole, user } = useAuth();
  const [showAlert, setShowAlert] = React.useState(false);

  if (!user) {
    return fallback || null;
  }

  const hasAccess = (permission && hasPermission(permission)) || 
                   (role && hasRole(role)) || 
                   (!permission && !role);

  if (!hasAccess) {
    return fallback || (
      <>
        <IonAlert
          isOpen={showAlert}
          onDidDismiss={() => setShowAlert(false)}
          header="Access Denied"
          message="You don't have permission to access this feature."
          buttons={['OK']}
        />
        <div 
          onClick={() => setShowAlert(true)}
          style={{ opacity: 0.5, pointerEvents: 'none' }}
        >
          {children}
        </div>
      </>
    );
  }

  return <>{children}</>;
};
```

**src/utils/pdfExport.ts**
```typescript
declare global {
  interface Window {
    jsPDF: any;
  }
}

export interface ExportOptions {
  filename?: string;
  format?: 'a4' | 'letter';
  orientation?: 'portrait' | 'landscape';
  includeHeader?: boolean;
  includeFooter?: boolean;
}

export class PDFExporter {
  private loadJsPDF(): Promise<void> {
    return new Promise((resolve, reject) => {
      if (window.jsPDF) {
        resolve();
        return;
      }

      const script = document.createElement('script');
      script.src = 'https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js';
      script.onload = () => resolve();
      script.onerror = () => reject(new Error('Failed to load jsPDF'));
      document.head.appendChild(script);
    });
  }

  async exportInvoiceToPDF(
    invoiceData: any, 
    options: ExportOptions = {}
  ): Promise<void> {
    try {
      await this.loadJsPDF();
      
      const { 
        filename = 'invoice.pdf',
        format = 'a4',
        orientation = 'portrait',
        includeHeader = true,
        includeFooter = true 
      } = options;

      const { jsPDF } = window;
      const doc = new jsPDF(orientation, 'mm', format);
      
      // Set up basic styling
      doc.setFontSize(12);
      let yPosition = 20;

      // Header
      if (includeHeader) {
        doc.setFontSize(20);
        doc.setFont(undefined, 'bold');
        doc.text('INVOICE', 20, yPosition);
        yPosition += 15;
      }

      // Invoice details
      doc.setFontSize(12);
      doc.setFont(undefined, 'normal');
      
      if (invoiceData.invoiceNumber) {
        doc.text(`Invoice #: ${invoiceData.invoiceNumber}`, 20, yPosition);
        yPosition += 8;
      }
      
      if (invoiceData.date) {
        doc.text(`Date: ${invoiceData.date}`, 20, yPosition);
        yPosition += 8;
      }

      yPosition += 10;

      // Bill To section
      doc.setFont(undefined, 'bold');
      doc.text('BILL TO:', 20, yPosition);
      yPosition += 8;
      
      doc.setFont(undefined, 'normal');
      if (invoiceData.billTo) {
        const billToLines = this.splitText(invoiceData.billTo, 80);
        billToLines.forEach(line => {
          doc.text(line, 20, yPosition);
          yPosition += 6;
        });
      }

      yPosition += 10;

      // From section
      doc.setFont(undefined, 'bold');
      doc.text('FROM:', 110, yPosition - (invoiceData.billTo ? 40 : 0));
      
      doc.setFont(undefined, 'normal');
      if (invoiceData.from) {
        const fromLines = this.splitText(invoiceData.from, 80);
        fromLines.forEach((line, index) => {
          doc.text(line, 110, yPosition - (invoiceData.billTo ? 32 : 0) + (index * 6));
        });
      }

      // Items table
      yPosition += 10;
      this.createItemsTable(doc, invoiceData.items || [], yPosition);

      // Footer
      if (includeFooter) {
        const pageHeight = doc.internal.pageSize.height;
        doc.setFontSize(10);
        doc.text('Thank you for your business!', 20, pageHeight - 20);
      }

      // Save the PDF
      doc.save(filename);
    } catch (error) {
      console.error('Failed to export PDF:', error);
      throw new Error('PDF export failed');
    }
  }

  private splitText(text: string, maxLength: number): string[] {
    const words = text.split(' ');
    const lines: string[] = [];
    let currentLine = '';

    words.forEach(word => {
      if ((currentLine + word).length <= maxLength) {
        currentLine += (currentLine ? ' ' : '') + word;
      } else {
        if (currentLine) lines.push(currentLine);
        currentLine = word;
      }
    });
    
    if (currentLine) lines.push(currentLine);
    return lines;
  }

  private createItemsTable(doc: any, items: any[], startY: number): void {
    const tableHeaders = ['Description', 'Quantity', 'Price', 'Amount'];
    const columnWidths = [80, 30, 30, 30];
    let yPos = startY;

    // Table headers
    doc.setFont(undefined, 'bold');
    doc.setFontSize(10);
    
    let xPos = 20;
    tableHeaders.forEach((header, index) => {
      doc.text(header, xPos, yPos);
      xPos += columnWidths[index];
    });
    
    yPos += 8;
    
    // Draw header line
    doc.line(20, yPos - 2, 190, yPos - 2);

    // Table rows
    doc.setFont(undefined, 'normal');
    items.forEach(item => {
      xPos = 20;
      
      doc.text(item.description || '', xPos, yPos);
      xPos += columnWidths[0];
      
      doc.text((item.quantity || '').toString(), xPos, yPos);
      xPos += columnWidths[1];
      
      doc.text((item.price || '').toString(), xPos, yPos);
      xPos += columnWidths[2];
      
      doc.text((item.amount || '').toString(), xPos, yPos);
      
      yPos += 8;
    });

    // Draw bottom line
    doc.line(20, yPos, 190, yPos);
  }

  async exportSpreadsheetToPDF(
    spreadsheetElement: HTMLElement,
    options: ExportOptions = {}
  ): Promise<void> {
    try {
      await this.loadJsPDF();
      
      const { filename = 'spreadsheet.pdf' } = options;
      const { jsPDF } = window;
      const doc = new jsPDF('landscape', 'mm', 'a4');

      // Convert HTML to canvas then to PDF
      const canvas = await this.htmlToCanvas(spreadsheetElement);
      const imgData = canvas.toDataURL('image/png');
      
      const imgWidth = 290;
      const pageHeight = 210;
      const imgHeight = (canvas.height * imgWidth) / canvas.width;
      let heightLeft = imgHeight;
      let position = 10;

      doc.addImage(imgData, 'PNG', 10, position, imgWidth, imgHeight);
      heightLeft -= pageHeight;

      while (heightLeft >= 0) {
        position = heightLeft - imgHeight + 10;
        doc.addPage();
        doc.addImage(imgData, 'PNG', 10, position, imgWidth, imgHeight);
        heightLeft -= pageHeight;
      }

      doc.save(filename);
    } catch (error) {
      console.error('Failed to export spreadsheet PDF:', error);
      throw new Error('Spreadsheet PDF export failed');
    }
  }

  private async htmlToCanvas(element: HTMLElement): Promise<HTMLCanvasElement> {
    // This would typically use html2canvas library
    // For now, creating a simple mock canvas
    const canvas = document.createElement('canvas');
    canvas.width = element.offsetWidth;
    canvas.height = element.offsetHeight;
    
    const ctx = canvas.getContext('2d');
    if (ctx) {
      ctx.fillStyle = 'white';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      
      // Add some basic text representation
      ctx.fillStyle = 'black';
      ctx.font = '12px Arial';
      ctx.fillText('Spreadsheet Export', 10, 30);
      ctx.fillText('(Full HTML to Canvas conversion requires html2canvas library)', 10, 50);
    }
    
    return canvas;
  }
}

export const pdfExporter = new PDFExporter();
```

**src/i18n/translations/en.ts**
```typescript
export const en = {
  common: {
    save: 'Save',
    cancel: 'Cancel',
    delete: 'Delete',
    edit: 'Edit',
    create: 'Create',
    export: 'Export',
    import: 'Import',
    loading: 'Loading...',
    error: 'Error',
    success: 'Success',
    warning: 'Warning',
    confirm: 'Confirm',
    yes: 'Yes',
    no: 'No',
    ok: 'OK',
    close: 'Close'
  },
  navigation: {
    home: 'Home',
    invoices: 'Invoices',
    settings: 'Settings',
    logout: 'Logout',
    profile: 'Profile'
  },
  invoice: {
    title: 'Invoice',
    number: 'Invoice Number',
    date: 'Invoice Date',
    dueDate: 'Due Date',
    billTo: 'Bill To',
    from: 'From',
    description: 'Description',
    quantity: 'Quantity',
    price: 'Price',
    amount: 'Amount',
    subtotal: 'Subtotal',
    tax: 'Tax',
    total: 'Total',
    create: 'Create Invoice',
    edit: 'Edit Invoice',
    delete: 'Delete Invoice',
    export: 'Export Invoice',
    exportPDF: 'Export to PDF',
    thankYou: 'Thank you for your business!'
  },
  auth: {
    login: 'Login',
    logout: 'Logout',
    email: 'Email',
    password: 'Password',
    forgotPassword: 'Forgot Password?',
    signUp: 'Sign Up',
    signIn: 'Sign In',
    invalidCredentials: 'Invalid credentials',
    accessDenied: 'Access Denied',
    noPermission: "You don't have permission to access this feature."
  },
  settings: {
    title: 'Settings',
    language: 'Language',
    theme: 'Theme',
    darkMode: 'Dark Mode',
    lightMode: 'Light Mode',
    notifications: 'Notifications',
    account: 'Account',
    security: 'Security',
    about: 'About'
  },
  roles: {
    admin: 'Administrator',
    manager: 'Manager',
    user: 'User',
    viewer: 'Viewer'
  },
  errors: {
    networkError: 'Network error occurred',
    serverError: 'Server error occurred',
    unexpectedError: 'An unexpected error occurred',
    exportFailed: 'Export failed',
    importFailed: 'Import failed'
  }
};
```

**src/i18n/translations/es.ts**
```typescript
export const es = {
  common: {
    save: 'Guardar',
    cancel: 'Cancelar',
    delete: 'Eliminar',
    edit: 'Editar',
    create: 'Crear',
    export: 'Exportar',
    import: 'Importar',
    loading: 'Cargando...',
    error: 'Error',
    success: 'Éxito',
    warning: 'Advertencia',
    confirm: 'Confirmar',
    yes: 'Sí',
    no: 'No',
    ok: 'OK',
    close: 'Cerrar'
  },
  navigation: {
    home: 'Inicio',
    invoices: 'Facturas',
    settings: 'Configuración',
    logout: 'Cerrar Sesión',
    profile: 'Perfil'
  },
  invoice: {
    title: 'Factura',
    number: 'Número de Factura',
    date: 'Fecha de Factura',
    dueDate: 'Fecha de Vencimiento',
    billTo: 'Facturar A',
    from: 'De',
    description: 'Descripción',
    quantity: 'Cantidad',
    price: 'Precio',
    amount: 'Importe',
    subtotal: 'Subtotal',
    tax: 'Impuesto',
    total: 'Total',
    create: 'Crear Factura',
    edit: 'Editar Factura',
    delete: 'Eliminar Factura',
    export: 'Exportar Factura',
    exportPDF: 'Exportar a PDF',
    thankYou: '¡Gracias por su negocio!'
  },
  auth: {
    login: 'Iniciar Sesión',
    logout: 'Cerrar Sesión',
    email: 'Correo Electrónico',
    password: 'Contraseña',
    forgotPassword: '¿Olvidaste tu contraseña?',
    signUp: 'Registrarse',
    signIn: 'Iniciar Sesión',
    invalidCredentials: 'Credenciales inválidas',
    accessDenied: 'Acceso Denegado',
    noPermission: 'No tienes permisos para acceder a esta función.'
  },
  settings: {
    title: 'Configuración',
    language: 'Idioma',
    theme: 'Tema',
    darkMode: 'Modo Oscuro',
    lightMode: 'Modo Claro',
    notifications: 'Notificaciones',
    account: 'Cuenta',
    security: 'Seguridad',
    about: 'Acerca de'
  },
  roles: {
    admin: 'Administrador',
    manager: 'Gerente',
    user: 'Usuario',
    viewer: 'Visualizador'
  },
  errors: {
    networkError: 'Error de red',
    serverError: 'Error del servidor',
    unexpectedError: 'Ocurrió un error inesperado',
    exportFailed: 'Error en la exportación',
    importFailed: 'Error en la importación'
  }
};
```

**src/i18n/translations/fr.ts**
```typescript
export const fr = {
  common: {
    save: 'Enregistrer',
    cancel: 'Annuler',
    delete: 'Supprimer',
    edit: 'Modifier',
    create: 'Créer',
    export: 'Exporter',
    import: 'Importer',
    loading: 'Chargement...',
    error: 'Erreur',
    success: 'Succès',
    warning: 'Avertissement',
    confirm: 'Confirmer',
    yes: 'Oui',
    no: 'Non',
    ok: 'OK',
    close: 'Fermer'
  },
  navigation: {
    home: 'Accueil',
    invoices: 'Factures',
    settings: 'Paramètres',
    logout: 'Déconnexion',
    profile: 'Profil'
  },
  invoice: {
    title: 'Facture',
    number: 'Numéro de Facture',
    date: 'Date de Facture',
    dueDate: 'Date d\'Échéance',
    billTo: 'Facturer À',
    from: 'De',
    description: 'Description',
    quantity: 'Quantité',
    price: 'Prix',
    amount: 'Montant',
    subtotal: 'Sous-total',
    tax: 'Taxe',
    total: 'Total',
    create: 'Créer une Facture',
    edit: 'Modifier la Facture',
    delete: 'Supprimer la Facture',
    export: 'Exporter la Facture',
    exportPDF: 'Exporter en PDF',
    thankYou: 'Merci pour votre entreprise!'
  },
  auth: {
    login: 'Connexion',
    logout: 'Déconnexion',
    email: 'E-mail',
    password: 'Mot de Passe',
    forgotPassword: 'Mot de passe oublié?',
    signUp: 'S\'inscrire',
    signIn: 'Se Connecter',
    invalidCredentials: 'Identifiants invalides',
    accessDenied: 'Accès Refusé',
    noPermission: 'Vous n\'avez pas la permission d\'accéder à cette fonctionnalité.'
  },
  settings: {
    title: 'Paramètres',
    language: 'Langue',
    theme: 'Thème',
    darkMode: 'Mode Sombre',
    lightMode: 'Mode Clair',
    notifications: 'Notifications',
    account: 'Compte',
    security: 'Sécurité',
    about: 'À Propos'
  },
  roles: {
    admin: 'Administrateur',
    manager: 'Gestionnaire',
    user: 'Utilisateur',
    viewer: 'Visualiseur'
  },
  errors: {
    networkError: 'Erreur de réseau',
    serverError: 'Erreur du serveur',
    unexpectedError: 'Une erreur inattendue s\'est produite',
    exportFailed: 'Échec de l\'exportation',
    importFailed: 'Échec de l\'importation'
  }
};
```

**src/i18n/index.ts**
```typescript
import { en } from './translations/en';
import { es } from './translations/es';
import { fr } from './translations/fr';

export type Language = 'en' | 'es' | 'fr';

export const translations = {
  en,
  es,
  fr
};

export const languages = [
  { code: 'en' as Language, name: 'English', flag: '🇺🇸' },
  { code: 'es' as Language, name: 'Español', flag: '🇪🇸' },
  { code: 'fr' as Language, name: 'Français', flag: '🇫🇷' }
];

export type TranslationKeys = typeof en;
export type NestedKeyOf<ObjectType extends object> = {
  [Key in keyof ObjectType & (string | number)]: ObjectType[Key] extends object
    ? `${Key}` | `${Key}.${NestedKeyOf<ObjectType[Key]>}`
    : `${Key}`;
}[keyof ObjectType & (string | number)];

export type TranslationKey = NestedKeyOf<TranslationKeys>;

class I18n {
  private currentLanguage: Language = 'en';
  private translations = translations;

  setLanguage(language: Language) {
    this.currentLanguage = language;
    localStorage.setItem('language', language);
  }

  getCurrentLanguage(): Language {
    return this.currentLanguage;
  }

  t(key: TranslationKey, params?: Record<string, string | number>): string {
    const keys = key.split('.');
    let translation: any = this.translations[this.currentLanguage];

    for (const k of keys) {
      translation = translation?.[k];
    }

    if (typeof translation !== 'string') {
      console.warn(`Translation not found for key: ${key}`);
      return key;
    }

    if (params) {
      return Object.keys(params).reduce((str, param) => {
        return str.replace(new RegExp(`{{${param}}}`, 'g'), params[param].toString());
      }, translation);
    }

    return translation;
  }

  init() {
    const savedLanguage = localStorage.getItem('language') as Language;
    if (savedLanguage && this.translations[savedLanguage]) {
      this.currentLanguage = savedLanguage;
    }
  }
}

export const i18n = new I18n();
```

**src/theme/ThemeContext.tsx**
```typescript
import React, { createContext, useContext, useState, useEffect } from 'react';

export type Theme = 'light' | 'dark';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setThemeState] = useState<Theme>('light');

  useEffect(() => {
    // Check for saved theme preference
    const savedTheme = localStorage.getItem('theme') as Theme;
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    
    const initialTheme = savedTheme || (prefersDark ? 'dark' : 'light');
    setThemeState(initialTheme);
    applyTheme(initialTheme);
  }, []);

  const applyTheme = (newTheme: Theme) => {
    document.body.classList.remove('light-theme', 'dark-theme');
    document.body.classList.add(`${newTheme}-theme`);
    
    // Update meta theme-color
    const metaThemeColor = document.querySelector('meta[name="theme-color"]');
    if (metaThemeColor) {
      metaThemeColor.setAttribute('content', newTheme === 'dark' ? '#1a1a1a' : '#ffffff');
    }
  };

  const setTheme = (newTheme: Theme) => {
    setThemeState(newTheme);
    localStorage.setItem('theme', newTheme);
    applyTheme(newTheme);
  };

  const toggleTheme = () => {
    const newTheme = theme === 'light' ? 'dark' : 'light';
    setTheme(newTheme);
  };

  const value: ThemeContextType = {
    theme,
    toggleTheme,
    setTheme,
  };

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error
```
