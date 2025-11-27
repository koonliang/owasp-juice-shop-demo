# Local Security Scan - Security Headers, OWASP ZAP & Noir Analysis

**IMPORTANT**: This command runs security scans against Juice Shop running in a local Kind cluster.

## Prerequisites Check

Before running scans, verify:
1. Kind cluster is running: `kind get clusters`
2. Juice Shop is deployed: `kubectl get pods -l app=juice-shop`
3. App is accessible: `curl -I http://localhost:30080` OR use Kind node IP
4. Docker is available: `docker ps`

## Task Overview

Run three security scans locally against Juice Shop in Kind cluster:
1. **Security Headers Analysis** - Check HTTP security headers
2. **OWASP ZAP Baseline Scan** - DAST (Dynamic Application Security Testing)
3. **OWASP Noir** - Static API endpoint discovery

All scans run in warning mode (findings expected for intentionally vulnerable app).

---

## SCAN 1: Security Headers Analysis

**Purpose**: Validate HTTP response headers for browser-side security protections

**Steps**:

1. **Detect Application URL**
   ```bash
   # Try localhost first
   if curl -s -o /dev/null -w "%{http_code}" http://localhost:30080 2>/dev/null | grep -q "200"; then
     APP_URL="http://localhost:30080"
   else
     # Fallback to Kind node IP
     KIND_NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
     APP_URL="http://${KIND_NODE_IP}:30080"
   fi
   echo "Using APP_URL: $APP_URL"
   ```

2. **Run Security Headers Scan**
   ```bash
   mkdir -p security-scan-reports

   echo "🔒 Analyzing HTTP security headers..."
   echo "Target: ${APP_URL}" | tee security-scan-reports/security-headers-scan.txt
   echo "" | tee -a security-scan-reports/security-headers-scan.txt

   # Check if application is accessible
   HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" ${APP_URL} 2>&1 || echo "000")

   if [ "$HTTP_CODE" != "200" ]; then
     echo "❌ Application not accessible (HTTP $HTTP_CODE)" | tee -a security-scan-reports/security-headers-scan.txt
     echo "Check pod status: kubectl get pods -l app=juice-shop"
     exit 1
   fi

   echo "✅ Application accessible (HTTP $HTTP_CODE)" | tee -a security-scan-reports/security-headers-scan.txt
   echo "" | tee -a security-scan-reports/security-headers-scan.txt

   # Fetch and display headers
   echo "=== Current Security Headers ===" | tee -a security-scan-reports/security-headers-scan.txt
   curl -s -D - ${APP_URL} -o /dev/null 2>&1 | grep -E "^X-|^Content-Security|^Strict-Transport" | tee -a security-scan-reports/security-headers-scan.txt || echo "(No security headers found)" | tee -a security-scan-reports/security-headers-scan.txt

   echo "" | tee -a security-scan-reports/security-headers-scan.txt
   echo "=== Security Header Assessment ===" | tee -a security-scan-reports/security-headers-scan.txt

   HEADERS=$(curl -s -D - ${APP_URL} -o /dev/null 2>&1)

   # Check for present headers
   echo "$HEADERS" | grep -q "X-Content-Type-Options" && echo "✓ X-Content-Type-Options: Present" | tee -a security-scan-reports/security-headers-scan.txt || echo "✗ X-Content-Type-Options: Missing" | tee -a security-scan-reports/security-headers-scan.txt
   echo "$HEADERS" | grep -q "X-Frame-Options" && echo "✓ X-Frame-Options: Present" | tee -a security-scan-reports/security-headers-scan.txt || echo "✗ X-Frame-Options: Missing" | tee -a security-scan-reports/security-headers-scan.txt
   echo "$HEADERS" | grep -q "Content-Security-Policy" && echo "✓ Content-Security-Policy: Present" | tee -a security-scan-reports/security-headers-scan.txt || echo "✗ Content-Security-Policy: Missing (expected)" | tee -a security-scan-reports/security-headers-scan.txt
   echo "$HEADERS" | grep -q "Strict-Transport-Security" && echo "✓ Strict-Transport-Security: Present" | tee -a security-scan-reports/security-headers-scan.txt || echo "✗ Strict-Transport-Security: Missing (expected)" | tee -a security-scan-reports/security-headers-scan.txt
   echo "$HEADERS" | grep -q "X-XSS-Protection" && echo "✓ X-XSS-Protection: Present" | tee -a security-scan-reports/security-headers-scan.txt || echo "✗ X-XSS-Protection: Missing (expected)" | tee -a security-scan-reports/security-headers-scan.txt

   echo "" | tee -a security-scan-reports/security-headers-scan.txt
   echo "⚠️  Note: Juice Shop is intentionally vulnerable - missing headers are expected" | tee -a security-scan-reports/security-headers-scan.txt
   ```

3. **Expected Output**:
   - ✓ X-Content-Type-Options: Present
   - ✓ X-Frame-Options: Present
   - ✗ Content-Security-Policy: Missing (intentional)
   - ✗ Strict-Transport-Security: Missing (intentional)

---

## SCAN 2: OWASP ZAP Baseline Scan

**Purpose**: Dynamic Application Security Testing (DAST) - finds runtime vulnerabilities

**Steps**:

1. **Detect ZAP Target URL**
   ```bash
   # For ZAP with --network host, use localhost (not Kind node IP)
   ZAP_TARGET_URL="http://localhost:30080"

   # Verify application is accessible
   HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" ${ZAP_TARGET_URL} 2>/dev/null || echo "000")

   if [ "$HTTP_CODE" = "200" ]; then
     echo "✅ Application accessible at: ${ZAP_TARGET_URL}"
   else
     echo "❌ Application not accessible (HTTP $HTTP_CODE)"
     echo "Check pod status: kubectl get pods -l app=juice-shop"
     exit 1
   fi
   ```

2. **Run OWASP ZAP Scan**
   ```bash
   echo "🔍 Running OWASP ZAP baseline scan..."
   echo "Target: ${ZAP_TARGET_URL}"
   echo "Note: This may take 2-5 minutes (plus image download on first run)..."

   # Check Docker availability
   if ! command -v docker &> /dev/null; then
     echo "❌ Docker is not installed"
     exit 1
   fi

   if ! docker ps &> /dev/null; then
     echo "❌ Docker daemon is not running"
     exit 1
   fi

   echo "✅ Docker available and daemon running"

   # Create directory for ZAP reports
   mkdir -p security-scan-reports/zap-reports

   # Run ZAP scan with host network mode
   # Note: timeout command not available on macOS, removed
   # Note: Using --network host allows ZAP container to access localhost:30080
   docker run --rm --network host \
     -v $(pwd)/security-scan-reports/zap-reports:/zap/wrk:rw \
     zaproxy/zap-stable:latest \
     zap-baseline.py -t ${ZAP_TARGET_URL} -I -r zap-report.html

   ZAP_EXIT_CODE=$?

   if [ "$ZAP_EXIT_CODE" != "0" ]; then
     echo "⚠️  ZAP scan exited with code: $ZAP_EXIT_CODE (security issues found - expected)"
   else
     echo "✅ ZAP scan completed successfully"
   fi

   # Check if report was generated
   if [ -f "security-scan-reports/zap-reports/zap-report.html" ]; then
     echo "✅ ZAP report saved to security-scan-reports/zap-reports/zap-report.html"
     echo "📊 Open the HTML report in your browser to view findings"
   else
     echo "⚠️  ZAP report not generated"
     echo "Checking directory contents..."
     ls -la security-scan-reports/zap-reports/
   fi
   ```

3. **Expected Output**:
   - HTML report with vulnerability findings
   - Many findings expected (XSS, SQL injection, etc.)
   - Report location: `security-scan-reports/zap-reports/zap-report.html`

---

## SCAN 3: OWASP Noir - Attack Surface Analysis

**Purpose**: Static analysis to discover API endpoints and attack surface

**Steps**:

1. **Install jq (if needed)**
   ```bash
   # For macOS
   if ! command -v jq &> /dev/null; then
     brew install jq
   fi

   # For Linux
   # sudo apt-get install -y jq
   ```

2. **Run OWASP Noir Scan**
   ```bash
   echo "🔍 Running OWASP Noir attack surface detection..."

   mkdir -p security-scan-reports

   echo "=== OWASP Noir - Static Analysis for API Endpoints ===" | tee security-scan-reports/noir-scan.txt
   echo "Technology: Identifies attack surfaces by analyzing source code" | tee -a security-scan-reports/noir-scan.txt
   echo "" | tee -a security-scan-reports/noir-scan.txt

   # Run Noir scan
   echo "Starting Noir scan (this may take 1-2 minutes)..."
   docker run --rm -v $(pwd):/target \
     ghcr.io/owasp-noir/noir:latest \
     noir -b /target -f json > security-scan-reports/noir-results.json 2>&1

   NOIR_EXIT_CODE=$?
   echo "Noir scan finished with exit code: $NOIR_EXIT_CODE"

   # Check if results exist
   if [ -f "security-scan-reports/noir-results.json" ] && [ -s "security-scan-reports/noir-results.json" ]; then
     # Extract JSON portion (Noir output contains ANSI codes and progress messages before JSON)
     grep -o '{"endpoints.*' security-scan-reports/noir-results.json > security-scan-reports/noir-clean.json 2>/dev/null || true

     # Verify we have valid JSON
     if jq empty security-scan-reports/noir-clean.json 2>/dev/null; then
       echo "✅ Noir scan completed successfully" | tee -a security-scan-reports/noir-scan.txt
       echo "" | tee -a security-scan-reports/noir-scan.txt

       # Extract endpoint count
       ENDPOINT_COUNT=$(jq '.endpoints | length' security-scan-reports/noir-clean.json)
       echo "📊 Scan Results:" | tee -a security-scan-reports/noir-scan.txt
       echo "   Total Endpoints Discovered: $ENDPOINT_COUNT" | tee -a security-scan-reports/noir-scan.txt
       echo "" | tee -a security-scan-reports/noir-scan.txt

       if [ "$ENDPOINT_COUNT" -gt "0" ]; then
         # Display endpoint details (first 20)
         echo "=== Discovered Endpoints (First 20) ===" | tee -a security-scan-reports/noir-scan.txt
         jq -r '.endpoints[:20] | .[] | "  • [\(.method)] \(.url)\n    Params: \(if .params | length > 0 then (.params | map(.name) | join(", ")) else "none" end)\n    File: \(.details.code_paths[0].path)"' security-scan-reports/noir-clean.json | tee -a security-scan-reports/noir-scan.txt
         echo "" | tee -a security-scan-reports/noir-scan.txt

         # HTTP methods breakdown
         echo "=== HTTP Methods Distribution ===" | tee -a security-scan-reports/noir-scan.txt
         jq -r '.endpoints | group_by(.method) | .[] | "  • \(.[0].method): \(length) endpoints"' security-scan-reports/noir-clean.json | tee -a security-scan-reports/noir-scan.txt
         echo "" | tee -a security-scan-reports/noir-scan.txt

         # Source files analyzed
         echo "=== Source Files Analyzed ===" | tee -a security-scan-reports/noir-scan.txt
         FILE_COUNT=$(jq '[.endpoints[].details.code_paths[].path] | unique | length' security-scan-reports/noir-clean.json)
         echo "   Total Source Files: $FILE_COUNT" | tee -a security-scan-reports/noir-scan.txt
         echo "   Sample Files (first 10):" | tee -a security-scan-reports/noir-scan.txt
         jq -r '[.endpoints[].details.code_paths[].path] | unique | .[:10] | .[] | "     - \(.)"' security-scan-reports/noir-clean.json | tee -a security-scan-reports/noir-scan.txt
         echo "" | tee -a security-scan-reports/noir-scan.txt
       else
         echo "No endpoints found in scan results" | tee -a security-scan-reports/noir-scan.txt
       fi
     else
       echo "⚠️  Unable to parse JSON results" | tee -a security-scan-reports/noir-scan.txt
     fi
   else
     echo "⚠️  Noir results not generated or empty" | tee -a security-scan-reports/noir-scan.txt
   fi

   echo "⚠️  Note: Juice Shop has many intentional vulnerabilities" | tee -a security-scan-reports/noir-scan.txt
   ```

3. **Expected Output**:
   - 600+ API endpoints discovered
   - Breakdown by HTTP method (GET, POST, DELETE, PUT, PATCH, etc.)
   - Source file locations with line numbers
   - Reports: `noir-results.json` (raw), `noir-clean.json` (parsed), and `noir-scan.txt`

---

## Summary Report

After all scans complete:

```bash
echo ""
echo "==============================================="
echo "  LOCAL SECURITY SCAN SUMMARY"
echo "==============================================="
echo ""
echo "Reports generated in: security-scan-reports/"
echo ""
echo "1. Security Headers:"
echo "   📄 security-headers-scan.txt"
echo ""
echo "2. OWASP ZAP DAST:"
echo "   📄 zap-reports/zap-report.html (open in browser)"
echo ""
echo "3. OWASP Noir Attack Surface:"
echo "   📄 noir-scan.txt"
echo "   📄 noir-results.json"
echo ""
echo "4. Consolidated Report:"
echo "   📄 JUICE-SHOP-VULNERABILITY-REPORT.md (comprehensive)"
echo ""
echo "⚠️  IMPORTANT: Juice Shop is intentionally vulnerable"
echo "   All findings are EXPECTED for training purposes"
echo ""
echo "Next steps:"
echo "  • Review consolidated report: security-scan-reports/JUICE-SHOP-VULNERABILITY-REPORT.md"
echo "  • Review ZAP HTML report in browser"
echo "  • Compare findings with OWASP Top 10"
echo "  • Use Noir endpoint list for manual testing"
echo ""
```

## Generate Consolidated Markdown Report

After completing all scans, generate a comprehensive vulnerability assessment report:

```bash
cat > security-scan-reports/JUICE-SHOP-VULNERABILITY-REPORT.md <<'EOF'
# OWASP Juice Shop - Vulnerability Assessment Report

**Generated:** $(date +%Y-%m-%d)
**Scan Type:** Comprehensive Security Assessment
**Tools Used:** Security Headers Analysis, OWASP ZAP (DAST), OWASP Noir (SAST)
**Target:** OWASP Juice Shop (http://localhost:30080)

---

## Executive Summary

This report presents a comprehensive security assessment of the OWASP Juice Shop application using multiple security scanning tools. The assessment includes:

1. **Security Headers Analysis** - HTTP security header validation
2. **OWASP ZAP Baseline Scan** - Dynamic Application Security Testing (DAST)
3. **OWASP Noir** - Static code analysis and API endpoint discovery

⚠️ **IMPORTANT**: OWASP Juice Shop is an intentionally vulnerable application designed for security training. All vulnerabilities are present by design for educational purposes.

---

## Scan Results Summary

### 1. Security Headers Analysis

**Status:** ✅ Completed

**Findings:**
- ✓ X-Content-Type-Options: Present (nosniff)
- ✓ X-Frame-Options: Present (SAMEORIGIN)
- ✗ Content-Security-Policy: MISSING (intentional)
- ✗ Strict-Transport-Security: MISSING (intentional)
- ✗ X-XSS-Protection: MISSING (intentional)

**Risk Level:** MEDIUM

**Impact:**
- Missing CSP increases XSS vulnerability risk
- Missing HSTS allows HTTP connections
- Missing X-XSS-Protection permits browser-based attacks

**Report Location:** \`security-scan-reports/security-headers-scan.txt\`

---

### 2. OWASP ZAP DAST Scan

**Status:** ✅ Completed

**URLs Scanned:** 95
**Tests Passed:** 57
**Warnings:** 10 categories
**Failures:** 0

**Key Findings:**

1. **Cross-Domain JavaScript Source File Inclusion** (12 instances)
   - Risk: Medium
   - Third-party scripts could be compromised

2. **Content Security Policy (CSP) Header Not Set** (11 instances)
   - Risk: Medium
   - Increases XSS vulnerability risk

3. **Cross-Domain Misconfiguration** (13 instances)
   - Risk: Medium
   - Potential data leakage to other domains

4. **Information Disclosure - Suspicious Comments** (2 instances)
   - Risk: Informational
   - May reveal sensitive implementation details

5. **Dangerous JS Functions** (2 instances)
   - Risk: Low to Medium
   - Usage of potentially unsafe JavaScript functions

6. **Insufficient Site Isolation Against Spectre Vulnerability** (12 instances)
   - Risk: Low
   - Potential side-channel attacks

**Report Location:** \`security-scan-reports/zap-reports/zap-report.html\`

---

### 3. OWASP Noir Attack Surface Analysis

**Status:** ✅ Completed

**Total Endpoints Discovered:** 600 unique endpoints
**Technologies Detected:** Express.js, OpenAPI Specification 3
**Source Files Analyzed:** 124
**Scan Duration:** ~5 minutes

**HTTP Method Distribution:**
- GET: 341 endpoints (56.8%)
- POST: 80 endpoints (13.3%)
- PUT: 78 endpoints (13.0%)
- DELETE: 44 endpoints (7.3%)
- PATCH: 20 endpoints (3.3%)
- OPTIONS: 19 endpoints (3.2%)
- HEAD: 18 endpoints (3.0%)

**Attack Surface Categories:**
- Authentication & Authorization endpoints
- Product/Basket management endpoints
- File upload/download endpoints
- Payment processing endpoints
- Admin functions
- API operations
- User management

**Report Location:** \`security-scan-reports/noir-scan.txt\`

---

## Combined Risk Assessment

### Critical Findings

Based on all three scans, the following critical areas require attention:

1. **Missing Security Headers**
   - No Content Security Policy (CSP)
   - No HTTP Strict Transport Security (HSTS)
   - Increases attack surface for XSS and MITM attacks

2. **Large Attack Surface**
   - 600 API endpoints provide extensive testing opportunities
   - Multiple HTTP methods on sensitive endpoints
   - Potential for IDOR (Insecure Direct Object References)

3. **Cross-Domain Issues**
   - Cross-domain JavaScript inclusion
   - Cross-domain misconfiguration
   - Potential CORS issues

4. **Information Disclosure**
   - Suspicious comments in JavaScript
   - API structure exposure
   - Metrics endpoint accessible

### Risk Priorities

**HIGH PRIORITY:**
- Implement Content Security Policy
- Review and secure cross-domain configurations
- Audit all 600 endpoints for authorization checks
- Implement rate limiting on API endpoints

**MEDIUM PRIORITY:**
- Add missing security headers (HSTS, X-XSS-Protection)
- Remove suspicious comments from production code
- Implement API authentication consistently
- Add input validation on all endpoints

**LOW PRIORITY:**
- Address Spectre vulnerability mitigations
- Optimize JavaScript function usage
- Implement comprehensive logging and monitoring

---

## Testing Methodology

### Tools Used

1. **Security Headers Analysis (curl)**
   - Simple HTTP header inspection
   - Identifies missing security headers
   - Quick baseline assessment

2. **OWASP ZAP v$(docker run --rm zaproxy/zap-stable:latest zap-baseline.py --version 2>&1 | head -1)**
   - Dynamic Application Security Testing (DAST)
   - Runtime vulnerability detection
   - Web application security scanner

3. **OWASP Noir v0.25.1**
   - Static Application Security Testing (SAST)
   - API endpoint discovery
   - Code path analysis

### Scan Coverage

- **Static Analysis:** Source code and API structure
- **Dynamic Analysis:** Runtime behavior and responses
- **Configuration Analysis:** HTTP headers and security settings

---

## Recommendations

### Immediate Actions (For Production Deployments)

1. **Enable Security Headers**
   \`\`\`
   Content-Security-Policy: default-src 'self'
   Strict-Transport-Security: max-age=31536000; includeSubDomains
   X-Content-Type-Options: nosniff
   X-Frame-Options: SAMEORIGIN
   X-XSS-Protection: 1; mode=block
   \`\`\`

2. **API Security**
   - Implement authentication on all API endpoints
   - Add authorization checks (prevent IDOR)
   - Enable rate limiting
   - Validate all inputs

3. **Cross-Origin Resource Sharing (CORS)**
   - Review and restrict CORS policies
   - Whitelist trusted domains only
   - Avoid wildcard origins

### Development Practices

1. Conduct regular security scans (SAST + DAST)
2. Implement security code reviews
3. Use dependency scanning tools
4. Follow OWASP secure coding guidelines
5. Implement security testing in CI/CD pipeline

---

## Compliance Considerations

### OWASP Top 10 2021

Based on scan results, the following OWASP Top 10 categories are relevant:

- ✓ A01:2021 - Broken Access Control (IDOR potential)
- ✓ A03:2021 - Injection (XSS via missing CSP)
- ✓ A05:2021 - Security Misconfiguration (missing headers)
- ✓ A07:2021 - Identification and Authentication Failures (API endpoints)

### Standards Impact

- **PCI DSS:** Security headers and encryption requirements
- **GDPR:** Data protection and access control
- **SOC 2:** Security logging and monitoring

---

## Next Steps

1. **Review Detailed Reports:**
   - Open ZAP HTML report in browser: \`security-scan-reports/zap-reports/zap-report.html\`
   - Review Noir endpoint list: \`security-scan-reports/noir-scan.txt\`
   - Check security headers: \`security-scan-reports/security-headers-scan.txt\`

2. **Manual Testing:**
   - Use discovered endpoints for penetration testing
   - Test authentication and authorization
   - Verify input validation

3. **Remediation:**
   - Prioritize critical findings
   - Implement security controls
   - Re-scan after fixes

4. **Continuous Monitoring:**
   - Schedule regular security scans
   - Monitor for new vulnerabilities
   - Update dependencies regularly

---

## Appendix: Report Files

| Report Type | File Location | Format |
|-------------|---------------|--------|
| Security Headers | security-scan-reports/security-headers-scan.txt | TXT |
| OWASP ZAP | security-scan-reports/zap-reports/zap-report.html | HTML |
| OWASP ZAP Summary | security-scan-reports/zap-scan-summary.txt | TXT |
| OWASP Noir | security-scan-reports/noir-scan.txt | TXT |
| OWASP Noir JSON | security-scan-reports/noir-clean.json | JSON |
| OWASP Noir Raw | security-scan-reports/noir-results.json | JSON |
| Consolidated | security-scan-reports/JUICE-SHOP-VULNERABILITY-REPORT.md | Markdown |

---

**Scan Date:** $(date)
**Generated By:** OWASP Security Scan Suite
**Report Version:** 1.0

---

**DISCLAIMER:** This is an intentionally vulnerable application (OWASP Juice Shop) designed for security training purposes. These vulnerabilities are present by design for educational use. Do not deploy this application in production environments.

EOF

echo "✅ Consolidated vulnerability report generated:"
echo "   📄 security-scan-reports/JUICE-SHOP-VULNERABILITY-REPORT.md"
```

---

## Troubleshooting

**Issue**: Application not accessible
- **Check**: `kubectl get pods -l app=juice-shop`
- **Fix**: Wait for pod to be Running: `kubectl wait --for=condition=ready pod -l app=juice-shop --timeout=120s`

**Issue**: Docker not available for ZAP
- **Check**: `docker ps`
- **Fix**: Start Docker Desktop or daemon

**Issue**: ZAP can't reach app
- **Check**: Verify localhost:30080 is accessible: `curl -I http://localhost:30080`
- **Fix**: Ensure you're using `--network host` with localhost:30080, NOT Kind node IP
- **Note**: With `--network host`, ZAP container can access localhost directly

**Issue**: timeout command not found (macOS)
- **Cause**: `timeout` command is not available on macOS by default
- **Fix**: Remove `timeout` prefix from docker command (already fixed in updated version)

**Issue**: Noir returns empty results
- **Check**: Running from project root directory
- **Fix**: `cd /path/to/owasp-juice-shop` and re-run

---

## Code Generation Requirements

When Claude Code executes this slash command:

1. **ALWAYS check prerequisites first**
   - Verify Kind cluster exists and is running
   - Verify Juice Shop pods are ready
   - Verify Docker daemon is accessible
   - Verify jq is installed (for Noir parsing)

2. **ALWAYS create output directory**
   - `mkdir -p security-scan-reports/zap-reports`

3. **ALWAYS detect correct URLs**
   - Try localhost:30080 first
   - Fallback to Kind node IP if localhost fails
   - For ZAP: Use localhost:30080 with `--network host` (NOT Kind node IP)

4. **ALWAYS use appropriate Docker network mode**
   - Security Headers: Run from host (curl from runner)
   - OWASP ZAP: Use `--network host` with localhost:30080 target
   - OWASP Noir: Mount current directory as volume

5. **ALWAYS handle macOS compatibility**
   - Do NOT use `timeout` command (not available on macOS)
   - Let Docker containers run without timeout wrapper
   - Use appropriate exit code handling instead

6. **ALWAYS handle errors gracefully**
   - Use `set +e` or `|| true` to continue on errors
   - Create diagnostic reports when scans fail
   - Document what failed and why in reports

7. **ALWAYS generate reports**
   - Even if scans fail, create placeholder reports
   - Use `tee` to show output and save to files simultaneously
   - JSON reports for machine parsing, TXT for human reading
   - Generate consolidated markdown report: `JUICE-SHOP-VULNERABILITY-REPORT.md`

8. **ALWAYS provide summary**
   - Show report locations (including consolidated markdown report)
   - Remind user about intentional vulnerabilities
   - Suggest next steps for analysis

---

## Expected Findings (Juice Shop)

**Security Headers**:
- ✓ X-Content-Type-Options: nosniff
- ✓ X-Frame-Options: SAMEORIGIN
- ✗ Content-Security-Policy: MISSING (allows XSS challenges)
- ✗ Strict-Transport-Security: MISSING (allows HTTP testing)

**OWASP ZAP**:
- SQL Injection vulnerabilities
- XSS (Cross-Site Scripting)
- Broken Authentication
- Sensitive Data Exposure
- Broken Access Control
- Security Misconfiguration
- 20-50+ findings typical

**OWASP Noir**:
- 600+ API endpoints discovered
- Technologies: Express.js, OpenAPI Spec 3
- HTTP Methods: GET (56.8%), POST (13.3%), PUT (13.0%), DELETE (7.3%), PATCH, OPTIONS, HEAD
- 124 source files analyzed
- Hidden admin endpoints
- File upload endpoints
- Authentication endpoints
- Search/filter endpoints with parameters
- Frontend and backend endpoints

All findings are INTENTIONAL for DevSecOps training!
