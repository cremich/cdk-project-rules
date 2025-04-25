# CDK Project Structure Rules

## File Naming Conventions

### General Naming Rules

- Use kebab-case for directory names: `data-storage/`, `search/`
- Use kebab-case for file names: `search-stack.ts`, `lambda-function.ts`
- Use PascalCase for class names: `SearchStack`, `LambdaFunction`
- Use camelCase for variable and function names: `createInstance`, `databaseConfig`

### Stack Files

- Name stack files with the `-stack` suffix: `search-stack.ts`
- Name construct files descriptively based on their purpose: `application-database.ts`, `api-gateway.ts`

### Test Files

- Name test files with the `.test.ts` suffix
- Match test file names to the files they are testing: `search-stack.test.ts`

## Folder Organization

```
app-name/
├── bin/                      # Entry point for CDK app
├── lib/                      # Main CDK constructs and stacks
│   ├── search/               # Contructs and stacks related to the search functionality
│   │   ├── search-stack.ts
│   │   └── constructs/
│   ├── website/              # Contructs and stacks related to the website
│   │   ├── website-stack.ts
│   │   └── constructs/
│   ├── auth/                 # Contructs and stacks related to the authentication functionality
│   │   ├── auth-stack.ts
│   │   └── constructs/
│   └── api/                  # Contructs and stacks related to the api functionality
│       ├── api-stack.ts
│       └── constructs/
├── common/                   # Shared constructs and utilities
│   ├── compute/              # Compute-related constructs (Lambda, ECS, etc.)
│   ├── storage/              # Storage-related constructs (S3, DynamoDB, etc.)
│   ├── network/              # Network-related constructs (VPC, subnets, etc.)
│   └── services/             # Service-specific constructs
├── config/                   # Environment-specific configuration
├── test/                     # Test files
└── utilities/                # Helper functions and scripts
```

- Group stacks and related constructs by its bounded contexts in dedicated directories under `lib/`
- Group constructs and utitlities shared across multiple bounded contexts in the `common/` folder.

## Separation of Logic and Configuration

### Configuration Management

- Store environment-specific configuration in the `config/` directory
- Use TypeScript interfaces to define configuration shapes
- Never hardcode environment-specific values in construct code

```typescript
// config/database-config.ts
export interface DatabaseConfig {
  instance: {
    instanceType: string;
    allocatedStorage: number;
    backupRetentionDays: number;
  };
  security: {
    allowedCidrs: string[];
  };
}
```

### Environment Context

- Use the CDK context to determine the name of the deployment environment
- Load the appropriate configuration based on the environment

```typescript
// lib/search/search-stack.ts
import { devDatabaseConfig, prodDatabaseConfig } from "../../config/database-config";

export class SearchStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const environmentName = this.node.tryGetContext("environmentName") || "dev";
    const config = environmentName === "prod" ? prodDatabaseConfig : devDatabaseConfig;

    // Use config to create resources
  }
}
```
