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

## Project Organization

### Service-Based Structure

```
service-name/              # One CDK app per deployable service
├── bin/                  # Entry point for CDK app
├── lib/                  # Main CDK constructs and stack
│   ├── constructs/      # Service-specific constructs
│   └── stack.ts         # Single stack for the service
├── test/                # Test files
└── utilities/           # Helper functions and scripts
```

- Create separate CDK apps for each independently deployable service
- Keep services small and focused on a single bounded context
- Shared resources (VPC, DNS zones) should be in their own CDK apps
- Use separate repositories for services that have independent lifecycles

## Stack Design

### Single Stack Approach

- Use a single stack per CDK app to maintain deployment atomicity
- Keep related resources together in one stack to preserve rollback capabilities
- Avoid splitting related resources across multiple stacks to prevent cross-stack reference issues
- Use constructs (L3) for logical grouping instead of multiple stacks
- Only create multiple stacks when:
  - Deploying to different regions
  - Exceeding CloudFormation resource limits (500 resources)
  - Working with truly independent shared resources (VPC, DNS zones)

### Environment Configuration

- Define environment configurations through stack properties
- Configure environments in the CDK app entry point (bin/)
- Avoid using CDK context or environment variables for configuration
- Synthesize all environment configurations simultaneously

Example:

```typescript
// bin/service.ts
const app = new cdk.App();

new ServiceStack(app, 'ServiceStackDev', {
  env: { account: '1234', region: 'us-east-1' },
  config: devConfig
});

new ServiceStack(app, 'ServiceStackProd', {
  env: { account: '5678', region: 'us-east-1' },
  config: prodConfig
});
```

### Stack Properties

- Define clear interfaces for stack configuration
- Pass environment-specific configuration through stack properties
- Keep all environment-specific values in the CDK app entry point

```typescript
// lib/stack.ts
export interface ServiceStackProps extends cdk.StackProps {
  config: {
    instanceType: string;
    capacity: number;
    // other configuration properties
  };
}

export class ServiceStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props: ServiceStackProps) {
    super(scope, id, props);
    // Use props.config to create resources
  }
}
```

## Best Practices

- Synthesize all environments at once to catch issues early
- Use the CDK assembly (cdk.out) for deployments instead of re-synthesizing
- Keep services small and focused to avoid stack splitting
- Use constructs for logical grouping of resources
- Maintain 1:1 relationships within a stack
- Move 1:N relationships to separate CDK apps
