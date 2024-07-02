Skip to content
Navigation Menu
tcaky
/
WikiMeThis

Type / to search
Code
Issues
1
Pull requests
Actions
Projects
1
Wiki
Security
Insights
Settings
Editing Mermaid graph example
 
 
Mermaid graph example
Write
Preview
    
Edit mode: 
Markdown
### Description of overall purpose of how we are currently using boards in the EC3 project.
 
```mermaid
graph TD;
A[EPIC] --> B[FEATURE]
B --> C[USER STORY]
C --> D[TASK]
E[High level grouping] --> F[Product]
F --> G[Work Type Group]
G --> H[Unit of work]
I[Operations] --> J[Governance]
J --> K[SOPs]
K --> L[Write SOP for monitoring SP search logs]
J --> M[Vendor Management]
M --> N[Renew A Vendor Contract]
```
### As above, but from Left to right (displays more complex hierarchy better):
```mermaid
graph LR;
A[EPIC] --> B[FEATURE]
B --> C[USER STORY]
C --> D[TASK]
E[High level grouping] --> F[Product]
F --> G[Work Type Group]
G --> H[Unit of work]
```
 
### Another example:
```mermaid
graph LR;
A[Operations] --> B[Ops-ProductA]
B --> C[Ops-ProductA-Change Requests]
C --> D[Update Views to filter better]
C --> E[Update workflow to email more people]
B --> F[Ops-ProductA-Communications]
F --> G[News at Company - Announce Release 1.0.1]
F --> H[News at Company- Announce Release 1.1.0]
```
 
### Last example:
```mermaid
graph LR;
A[Operations] --> B[Ops-General]
B --> C[Ops-General-Infrastructure]
C --> D[Replace physical servers with VMs]
B --> E[Ops-General-Certificates]
E --> F[Get Cert for Server 001]
E --> G[Get Cert for Server 002]
E --> H[Get Cert for Server 003]
E --> I[Get Cert for Server 004]
```

No file chosen
Attach files by dragging & dropping, selecting or pasting them.
Edit message
Write a small message here explaining this change. (Optional)
Footer
© 2024 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
Status
Docs
Contact
Manage cookies
Do not share my personal information
Editing Mermaid graph example · tcaky/WikiMeThis Wiki