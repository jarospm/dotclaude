---
name: start-node
description: Initialize a minimal Node.js/TypeScript project with modern tooling
disable-model-invocation: true
argument-hint: [project-path]
allowed-tools: Bash(npm:*), Bash(mkdir:*), Write, Edit
---

# Initialize Minimal Node.js/TypeScript Project

Set up a new Node.js project with TypeScript at `$ARGUMENTS` (or current directory if not specified).

## Steps

1. **Initialize npm** — run `npm init -y`

2. **Install dev dependencies**
   ```bash
   npm install --save-dev typescript @types/node tsx
   ```

3. **Create tsconfig.json** with minimal Node settings:
   ```json
   {
     "compilerOptions": {
       "rootDir": "./src",
       "outDir": "./dist",
       "module": "nodenext",
       "target": "esnext",
       "types": ["node"],
       "strict": true,
       "skipLibCheck": true
     },
     "include": ["src/**/*"]
   }
   ```

4. **Update package.json**:
   - Set `"type": "module"` for ESM
   - Set `"main": "dist/index.js"`
   - Add scripts:
     ```json
     "scripts": {
       "build": "tsc",
       "dev": "tsx src/index.ts"
     }
     ```
   - Remove empty/placeholder fields (description, keywords, author, test script)

5. **Create .gitignore**:
   ```
   # OS
   .DS_Store

   # Dependencies
   node_modules/

   # Build output
   dist/

   # Secrets
   .env
   ```

6. **Create src/index.ts** with hello world:
   ```typescript
   console.log("Hello, world!");
   ```

7. **Create README.md** with setup and dev commands

## Final structure

```
project/
├── src/
│   └── index.ts
├── .gitignore
├── package.json
├── package-lock.json
├── tsconfig.json
└── README.md
```

## Usage

```
/start-node ./my-project
/start-node                  # use current directory
```
