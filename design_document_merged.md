# Personalized Tarot Reading Bot Design Document
*Comprehensive Technical Specification and Implementation Guide*

[Previous content remains unchanged until Section 2.4]

### 2.4 Project Structure and Organization

#### Core Project Structure
```
/tarot-bot
├── src/                    # Source code
│   ├── api/               # API endpoints and controllers
│   ├── core/              # Core business logic
│   ├── db/                # Database models and migrations
│   ├── services/          # Business services
│   └── config/            # Configuration files
│       └── limits.yaml    # System constraints and thresholds
├── docs/                  # Project documentation
│   ├── api/              # API documentation and references
│   ├── guides/           # User and developer guides
│   └── technical/        # Technical specifications
├── support/              # Support utilities and tools
│   ├── scripts/          # Maintenance and utility scripts
│   ├── tools/            # Helper tools and utilities
│   └── maintenance/      # Maintenance related utilities
├── tests/                # Test files
└── config/               # Environment and app configuration
```

#### System Constraints (limits.yaml)
```yaml
# src/config/limits.yaml
api:
  rate_limits:
    default:
      requests_per_minute: 60
      requests_per_hour: 1000
    premium:
      requests_per_minute: 120
      requests_per_hour: 2000

performance:
  max_concurrent_readings: 100
  max_response_time_ms: 2000
  cache_ttl_seconds: 3600
  max_queue_size: 1000

security:
  max_failed_logins: 5
  session_timeout_minutes: 30
  password_min_length: 8
  max_session_per_user: 3

resources:
  max_memory_usage_mb: 2048
  max_cpu_usage_percent: 80
  max_disk_usage_gb: 10
  max_database_connections: 100

business:
  max_daily_readings: 50
  max_credits_per_purchase: 10000
  min_credits_per_purchase: 10
  max_refund_window_days: 30
```

#### Documentation Structure (/docs)

##### API Documentation (/docs/api)
```
/docs/api/
├── endpoints/                 # Detailed API endpoint documentation
│   ├── authentication.md     # Authentication endpoints
│   ├── readings.md          # Reading-related endpoints
│   ├── users.md            # User management endpoints
│   └── payments.md         # Payment processing endpoints
├── guides/                   # Integration guides
│   ├── getting-started.md   # Quick start guide
│   ├── authentication.md    # Authentication guide
│   └── best-practices.md    # API usage best practices
└── reference/               # API reference materials
    ├── errors.md           # Error codes and handling
    ├── rate-limits.md      # Rate limiting details
    └── schemas.md          # Data schemas and models
```

##### User and Developer Guides (/docs/guides)
```
/docs/guides/
├── user/                     # User documentation
│   ├── getting-started/     
│   │   ├── installation.md  # Installation guide
│   │   ├── configuration.md # Configuration guide
│   │   └── first-steps.md   # Initial setup and usage
│   ├── features/           
│   │   ├── readings.md      # Reading features guide
│   │   ├── payments.md      # Payment system guide
│   │   └── profiles.md      # User profile management
│   └── troubleshooting/    
│       ├── common-issues.md # Common problems and solutions
│       └── faq.md          # Frequently asked questions
├── developer/               # Developer documentation
│   ├── setup/             
│   │   ├── environment.md   # Development environment setup
│   │   ├── dependencies.md  # Managing dependencies
│   │   └── configuration.md # Development configuration
│   ├── guidelines/        
│   │   ├── coding-style.md  # Coding standards
│   │   ├── testing.md       # Testing guidelines
│   │   └── security.md      # Security best practices
│   └── deployment/        
│       ├── preparation.md   # Deployment preparation
│       ├── process.md       # Deployment process
│       └── monitoring.md    # Post-deployment monitoring
└── contributing/            # Contribution guidelines
    ├── process.md          # Contribution process
    ├── standards.md        # Code and documentation standards
    └── reviews.md          # Code review guidelines
```

##### Technical Specifications (/docs/technical)
```
/docs/technical/
├── architecture/            # System architecture documentation
│   ├── overview.md         # High-level architecture
│   ├── components.md       # Component specifications
│   └── interfaces.md       # Interface definitions
├── database/               # Database documentation
│   ├── schema.md          # Database schema
│   ├── migrations.md       # Migration procedures
│   └── optimization.md     # Performance optimization
├── security/               # Security documentation
│   ├── authentication.md   # Authentication system
│   ├── authorization.md    # Authorization rules
│   └── encryption.md       # Data encryption
├── performance/            # Performance documentation
│   ├── monitoring.md       # Performance monitoring
│   ├── optimization.md     # Optimization strategies
│   └── scaling.md         # Scaling procedures
└── integration/            # Integration documentation
    ├── apis.md            # External API integration
    ├── services.md        # Service integration
    └── webhooks.md        # Webhook implementation
```

#### Support Utilities (/support)

##### Maintenance Scripts (/support/scripts)
- Database backup/restore utilities
- Log rotation and management
- Cache cleanup and optimization
- Performance monitoring tools
- Health check scripts
- Data migration utilities

##### Helper Tools (/support/tools)
- Development utilities
- Testing helpers and frameworks
- Debugging tools and profilers
- Code generators
- Documentation generators
- Configuration validators

##### Maintenance Utilities (/support/maintenance)
- System diagnostics tools
- Resource monitoring utilities
- Error tracking systems
- Performance profiling tools
- Security auditing utilities
- Backup verification tools

[Rest of the original document content continues unchanged...]
