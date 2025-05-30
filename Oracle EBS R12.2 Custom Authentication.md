# Oracle EBS R12.2 Custom Authentication Implementation Guide

## Overview
This guide provides step-by-step instructions for implementing custom authentication code for Oracle E-Business Suite (EBS) R12.2 login page, including modifying the Login.js file and managing access to old login pages.

## Architecture Overview

### Key Components
- **AppsLocalLogin.jsp**: The main JSP file referenced during EBS login
- **Login.js**: JavaScript file containing login logic and form validation
- **submitCredentials()**: Primary method handling login form submission
- **APPS_LOCAL_LOGIN_URL**: Profile option controlling default login page in R12.2
- **APPS_LOGIN_FUNCTION**: Alternative profile option for login page configuration

## Implementation Steps

### Step 1: Backup Existing Files

Before making any modifications, create backups of existing files:

```bash
# Navigate to OA_HTML directory
cd $OA_HTML

# Create backup directory
mkdir -p backups/$(date +%Y%m%d)

# Backup original files
cp login.js backups/$(date +%Y%m%d)/login.js.backup
cp AppsLocalLogin.jsp backups/$(date +%Y%m%d)/AppsLocalLogin.jsp.backup
```

### Step 2: Locate and Access Files

#### File Locations:
```bash
# Login.js location
$OA_HTML/login.js

# AppsLocalLogin.jsp location
$OA_HTML/AppsLocalLogin.jsp

# Alternative locations (if customized)
$JAVA_TOP/oracle/apps/fnd/sso/login/webui/
```

### Step 3: Modify Login.js File

#### Original submitCredentials() Method Structure:
```javascript
function submitCredentials() {
    // Original EBS login logic
    var username = document.forms[0].username.value;
    var password = document.forms[0].password.value;
    
    // Standard validation and submission
    if (validateCredentials()) {
        document.forms[0].submit();
    }
}
```

#### Custom Authentication Enhancement:
```javascript
// Add custom authentication logic to Login.js
function submitCredentials() {
    // Get form values
    var username = document.forms[0].username.value;
    var password = document.forms[0].password.value;
    var responsibility = document.forms[0].responsibility ? document.forms[0].responsibility.value : '';
    
    // Custom validation logic
    if (!customValidateCredentials(username, password)) {
        return false;
    }
    
    // Custom authentication preprocessing
    if (!preprocessAuthentication(username)) {
        return false;
    }
    
    // Call original submission logic
    return originalSubmitCredentials();
}

// Custom validation function
function customValidateCredentials(username, password) {
    // Add your custom validation logic here
    if (username.length < 3) {
        alert('Username must be at least 3 characters long');
        return false;
    }
    
    // Custom password policy validation
    if (!validatePasswordPolicy(password)) {
        alert('Password does not meet security requirements');
        return false;
    }
    
    // Additional custom checks
    if (!checkUserAccess(username)) {
        alert('User access restricted');
        return false;
    }
    
    return true;
}

// Custom password policy validation
function validatePasswordPolicy(password) {
    // Implement your password policy
    var minLength = 8;
    var hasUpperCase = /[A-Z]/.test(password);
    var hasLowerCase = /[a-z]/.test(password);
    var hasNumbers = /\d/.test(password);
    var hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);
    
    return password.length >= minLength && hasUpperCase && hasLowerCase && hasNumbers && hasSpecialChar;
}

// Custom user access check
function checkUserAccess(username) {
    // Add custom logic to check user access
    // This could involve AJAX calls to custom APIs
    var restrictedUsers = ['test', 'demo', 'guest'];
    return !restrictedUsers.includes(username.toLowerCase());
}

// Preprocessing before authentication
function preprocessAuthentication(username) {
    // Log authentication attempt
    logAuthenticationAttempt(username);
    
    // Additional preprocessing logic
    return true;
}

// Authentication logging
function logAuthenticationAttempt(username) {
    // Custom logging logic
    var timestamp = new Date().toISOString();
    console.log('Login attempt for user: ' + username + ' at ' + timestamp);
    
    // You can extend this to send logs to server
    // sendLogToServer(username, timestamp);
}

// Preserve original function
var originalSubmitCredentials = submitCredentials;
```

### Step 4: Configure Profile Options

#### Set APPS_LOGIN_FUNCTION Profile Option:

```sql
-- Connect as APPS user
-- Update profile option to use old login page

-- Method 1: Using SQL
UPDATE fnd_profile_option_values 
SET profile_option_value = 'APPS_LOGIN_FUNCTION'
WHERE profile_option_id = (
    SELECT profile_option_id 
    FROM fnd_profile_options 
    WHERE profile_option_name = 'APPS_LOCAL_LOGIN_URL'
)
AND level_id = 10001; -- Site level

COMMIT;

-- Method 2: Using Function
BEGIN
    fnd_profile.save('APPS_LOCAL_LOGIN_URL', 'APPS_LOGIN_FUNCTION', 'SITE');
    COMMIT;
END;
/
```

#### Alternative Configuration via System Administrator:
1. Login to EBS as System Administrator
2. Navigate to: **System Administrator → Profile → System**
3. Search for profile: **APPS_LOCAL_LOGIN_URL**
4. Set value to: **APPS_LOGIN_FUNCTION**
5. Save changes

### Step 5: Modify AppsLocalLogin.jsp (Optional)

If you need to modify the JSP file itself:

```jsp
<%-- Add custom HTML/JavaScript at the beginning --%>
<script type="text/javascript">
// Custom JavaScript for login page
function initCustomLogin() {
    // Add custom initialization logic
    document.addEventListener('DOMContentLoaded', function() {
        // Custom DOM manipulation
        addCustomValidation();
        setupCustomEventHandlers();
    });
}

function addCustomValidation() {
    // Add real-time validation
    var usernameField = document.getElementById('username');
    var passwordField = document.getElementById('password');
    
    if (usernameField) {
        usernameField.addEventListener('blur', function() {
            validateUsernameRealtime(this.value);
        });
    }
    
    if (passwordField) {
        passwordField.addEventListener('input', function() {
            checkPasswordStrength(this.value);
        });
    }
}

// Initialize custom login functionality
initCustomLogin();
</script>

<%-- Rest of original JSP content --%>
```

### Step 6: Testing and Validation

#### Test Plan:
1. **Backup Verification**: Ensure all backups are created
2. **Login Functionality**: Test standard login process
3. **Custom Validation**: Verify custom validation rules work
4. **Error Handling**: Test error scenarios
5. **Profile Option**: Confirm profile option changes take effect
6. **Browser Compatibility**: Test across different browsers

#### Testing Script:
```javascript
// JavaScript test functions to add to login page for testing
function testCustomAuthentication() {
    console.log('Testing custom authentication...');
    
    // Test cases
    var testCases = [
        {username: 'test', password: 'weak', shouldFail: true},
        {username: 'validuser', password: 'StrongPass123!', shouldFail: false},
        {username: 'ab', password: 'StrongPass123!', shouldFail: true}
    ];
    
    testCases.forEach(function(testCase, index) {
        console.log('Test case ' + (index + 1) + ':', testCase);
        var result = customValidateCredentials(testCase.username, testCase.password);
        console.log('Result:', result, 'Expected to fail:', testCase.shouldFail);
    });
}

// Run tests (remove in production)
// testCustomAuthentication();
```

## Security Considerations

### Best Practices:
1. **Input Validation**: Always validate user input on both client and server side
2. **Password Security**: Implement strong password policies
3. **Session Management**: Ensure proper session handling
4. **Logging**: Log authentication attempts for security monitoring
5. **Error Messages**: Don't reveal too much information in error messages

### Security Enhancements:
```javascript
// Enhanced security features
function enhancedSecurityFeatures() {
    // Implement CAPTCHA for multiple failed attempts
    var failedAttempts = getFailedAttempts();
    if (failedAttempts >= 3) {
        showCaptcha();
    }
    
    // Implement account lockout
    if (failedAttempts >= 5) {
        lockAccount();
        return false;
    }
    
    return true;
}

// Track failed login attempts
function trackFailedAttempt(username) {
    var attempts = localStorage.getItem('failed_' + username) || 0;
    attempts++;
    localStorage.setItem('failed_' + username, attempts);
    
    // Set expiry for lockout
    var lockoutExpiry = new Date();
    lockoutExpiry.setMinutes(lockoutExpiry.getMinutes() + 30);
    localStorage.setItem('lockout_' + username, lockoutExpiry.toISOString());
}
```

## Troubleshooting

### Common Issues:

#### 1. Login Page Not Loading
```bash
# Check file permissions
chmod 644 $OA_HTML/login.js
chmod 644 $OA_HTML/AppsLocalLogin.jsp

# Restart Apache
$ADMIN_SCRIPTS_HOME/adapcctl.sh stop
$ADMIN_SCRIPTS_HOME/adapcctl.sh start
```

#### 2. JavaScript Errors
- Check browser console for errors
- Validate JavaScript syntax
- Ensure all required functions are defined

#### 3. Profile Option Not Taking Effect
```sql
-- Verify profile option setting
SELECT fpov.profile_option_value, 
       fpo.profile_option_name,
       fpov.level_value
FROM fnd_profile_option_values fpov,
     fnd_profile_options fpo
WHERE fpov.profile_option_id = fpo.profile_option_id
AND fpo.profile_option_name = 'APPS_LOCAL_LOGIN_URL';
```

#### 4. Cache Issues
```bash
# Clear Apache cache
rm -rf $IAS_ORACLE_HOME/j2ee/oacore/persistence/*
rm -rf $IAS_ORACLE_HOME/j2ee/oafm/persistence/*

# Restart services
$ADMIN_SCRIPTS_HOME/adstpall.sh
$ADMIN_SCRIPTS_HOME/adstrtal.sh
```

## Rollback Procedures

### Emergency Rollback:
```bash
# Restore original files
cd $OA_HTML
cp backups/$(date +%Y%m%d)/login.js.backup login.js
cp backups/$(date +%Y%m%d)/AppsLocalLogin.jsp.backup AppsLocalLogin.jsp

# Reset profile option
sqlplus apps/<password> << EOF
UPDATE fnd_profile_option_values 
SET profile_option_value = 'APPS_LOCAL_LOGIN_URL'
WHERE profile_option_id = (
    SELECT profile_option_id 
    FROM fnd_profile_options 
    WHERE profile_option_name = 'APPS_LOCAL_LOGIN_URL'
)
AND level_id = 10001;
COMMIT;
EXIT;
EOF

# Restart Apache
$ADMIN_SCRIPTS_HOME/adapcctl.sh restart
```

## Maintenance and Monitoring

### Regular Maintenance Tasks:
1. **Log Review**: Regularly review authentication logs
2. **Security Updates**: Keep custom code updated with security patches
3. **Performance Monitoring**: Monitor login page performance
4. **Backup Management**: Maintain current backups of customizations

### Monitoring Script:
```bash
#!/bin/bash
# login_monitor.sh - Monitor login page functionality

LOG_FILE="/tmp/login_monitor.log"
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

# Check if login page is accessible
if curl -s -o /dev/null -w "%{http_code}" http://your-ebs-server/OA_HTML/AppsLocalLogin.jsp | grep -q "200"; then
    echo "[$TIMESTAMP] Login page accessible" >> $LOG_FILE
else
    echo "[$TIMESTAMP] ERROR: Login page not accessible" >> $LOG_FILE
    # Send alert email
    mail -s "EBS Login Page Alert" admin@company.com < $LOG_FILE
fi
```

## Conclusion

This implementation provides a comprehensive approach to customizing Oracle EBS R12.2 authentication while maintaining system security and functionality. Always test thoroughly in a development environment before applying to production systems.

### Key Points to Remember:
- Always backup original files before modifications
- Test all changes thoroughly
- Monitor system performance after implementation
- Maintain proper security practices
- Document all customizations for future reference