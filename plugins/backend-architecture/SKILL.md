---
name: Designing Backend Architecture
description: Backend system architecture covering database design (UUIDv7/bigint IDs), Redis/Valkey setup, configuration management, and caching strategies. Use when designing scalable backends, configuring Redis instances, managing environment variables, or when user mentions UUID, Redis, Valkey, cache, session, ActionCable, ENV, or configuration.
---

# Backend Architecture

## Overview

Best practices focused on backend system architecture design, covering data storage strategies, configuration management, and system design patterns. These guidelines are primarily for Ruby on Rails applications, but many architectural principles apply to other backend frameworks.

## Redis/Valkey Architecture

Proper Redis architecture is the foundation of high-performance applications. See [Redis Architecture Guide](references/redis-architecture.md) for:

### Three-Instance Separation

Why you need three separate Redis instances instead of a single instance.

- **redis_cache (Cache)**
  - **Purpose**: General application cache (fragment cache, query cache)
  - **Eviction policy**: `allkeys-lru` (automatically evict old data when memory is full)
  - **Persistence**: No persistence (restart clears without impact)
  - **Characteristics**: Can tolerate data loss

- **redis_session (Session/Token)**
  - **Purpose**: User sessions, access tokens, critical temporary data
  - **Eviction policy**: `noeviction` (reject writes when memory is full, don't evict data)
  - **Persistence**: AOF persistence (appendonly.aof)
  - **Characteristics**: Cannot lose data

- **redis_cable (ActionCable)**
  - **Purpose**: WebSocket connections, real-time message pushing
  - **Eviction policy**: `noeviction`
  - **Persistence**: AOF persistence
  - **Characteristics**: High concurrent pub/sub

### Why Separate Instances?

- **Prevent Cache from Evicting Sessions/Tokens**
  - Cache uses LRU to evict old data
  - If sessions are evicted → users forced to logout
  - Separation keeps them independent

- **Performance Isolation**
  - High cache traffic doesn't affect session read/write
  - ActionCable pub/sub doesn't block cache queries
  - Independent memory quotas

- **Different Persistence Strategies**
  - Cache doesn't need persistence (clearing on restart is acceptable)
  - Session/cable need AOF persistence (avoid data loss)

### Docker Compose Configuration

- **Three Independent Redis Containers**
  - Different port mappings
  - Different volume mounts
  - Different redis.conf

- **Connection String Management**
  - `REDIS_CACHE_URL`
  - `REDIS_SESSION_URL`
  - `REDIS_CABLE_URL`

- **Health Checks**
  - Regular ping to ensure availability
  - Auto-restart failed containers

### Connection Pool Best Practices

- **Pool Size Configuration**
  - General recommendation: Rails threads * 1.5
  - Consider Sidekiq worker count
  - Monitor connection usage

- **Timeout Configuration**
  - Connect timeout
  - Read/Write timeout
  - Pool checkout timeout

## Configuration Management

Secure and flexible configuration management strategies. See [Configuration Guide](references/configuration.md) for:

### Configuration Layers

Three tiers of application configuration.

- **Environment Variables (ENV)**
  - **Purpose**: Runtime environment settings (database URL, Redis URL)
  - **Loading**: `.env` files (dotenv gem)
  - **Security**: Don't commit to git (`.gitignore`)
  - **Examples**: `DATABASE_URL`, `REDIS_URL`, `RAILS_ENV`

- **Docker Secrets**
  - **Purpose**: Highly sensitive information (API keys, passwords)
  - **Loading**: Mounted to `/run/secrets/`
  - **Security**: File permissions 600, encrypted storage
  - **Examples**: `STRIPE_SECRET_KEY`, `JWT_SECRET`

- **Settings (config gem)**
  - **Purpose**: Application logic configuration (feature flags, business constants)
  - **Loading**: YAML files (`config/settings.yml`)
  - **Security**: Can commit to git (no sensitive info)
  - **Examples**: `Settings.max_upload_size`, `Settings.features.enabled`

### Configuration Best Practices

- **Sensitive Information Handling**
  - Never commit `.env` to git
  - Use `.env.example` as template
  - Docker Secrets use 600 permissions
  - Regularly rotate API keys

- **Environment Differentiation**
  - Development: `.env.development`
  - Test: `.env.test`
  - Production: Docker Secrets + ENV

- **Default Value Strategy**
  - ENV variables provide sensible defaults
  - Settings have explicit fallbacks
  - Validate required configuration exists

### Rails Credentials (Alternative)

Rails built-in encrypted configuration management.

- **Advantages**
  - Encrypted storage, can commit to git
  - Native Rails support
  - Environment separation (development, production)

- **Disadvantages**
  - Need to manage master.key
  - Modifications require decrypt → edit → encrypt
  - Not suitable for frequently changing config

- **Use Cases**
  - Small projects
  - Not using Docker
  - Infrequent configuration changes

## Database Design

### Primary Key Strategy: UUIDv7 vs bigint

Choose database primary key types based on security, performance, and scalability needs.

**Default recommendation: UUIDv7**
- ✅ Prevents business intelligence leakage (no sequential IDs)
- ✅ Same performance as bigint (time-ordered, append-only B-tree)
- ✅ Native PostgreSQL 18+ / MySQL 8.4+ support
- ✅ Globally unique for distributed systems
- ⚠️ Larger storage (16 bytes vs 8 bytes)

**When to use bigint:**
- Internal tables not exposed via API
- Join tables
- High-volume logging tables
- Tables with extreme storage constraints

**Security principle:**

UUIDs prevent enumeration attacks but always implement proper authorization:

```ruby
# ✅ Correct: UUID + authorization
def show
  @order = current_user.orders.find(params[:id])
end

# ❌ Wrong: Relying only on UUID secrecy
def show
  @order = Order.find(params[:id])
end
```

**For comprehensive coverage including:**
- Business intelligence leakage examples and risks
- Performance benchmarks (UUIDv7 vs UUIDv4 vs bigint vs ULID)
- Framework-specific implementation (Rails, Django, Node.js)
- Migration strategies for existing projects
- Index optimization for UUID foreign keys
- URL display format options

**See [Database ID Strategy Guide](references/database-id-strategy.md)**

## Architecture Patterns

### Caching Strategies

- **Fragment Caching**
  - Cache view fragments
  - Use redis_cache instance
  - Automatic expiration and eviction

- **Query Caching**
  - Cache database query results
  - Use `Rails.cache`
  - Mind cache invalidation strategy

- **HTTP Caching**
  - CDN caches static assets
  - ETag / Last-Modified
  - Cloudflare integration

### Session Management

- **Token-based Authentication**
  - Store in redis_session
  - Fast lookup
  - Immediate revocation
  - See: [web-security skill - Authentication Guide]

### Real-time Communication

- **ActionCable Architecture**
  - redis_cable as pub/sub backend
  - WebSocket connection management
  - Message broadcast strategy
  - See: [deployment-practices skill - Cloudflare Tunnel Guide]

## System Design Checklist

Checklist when designing backend systems:

### Redis Architecture
- [ ] Separate cache, session, cable instances
- [ ] Configure correct eviction policy
- [ ] Set appropriate persistence strategy
- [ ] Monitor memory usage
- [ ] Set maxmemory limits

### Configuration Management
- [ ] Add `.env` to `.gitignore`
- [ ] Provide `.env.example` template
- [ ] Use correct permissions for Docker Secrets
- [ ] Validate required environment variables exist
- [ ] Document all configuration options

### Performance
- [ ] Configure appropriate connection pool size
- [ ] Set timeout parameters
- [ ] Monitor connection usage
- [ ] Implement caching strategies
- [ ] Regularly clean expired data

### Security
- [ ] Don't commit sensitive info to git
- [ ] Use HTTPS for transmission
- [ ] Regularly rotate secrets
- [ ] Restrict Redis network access
- [ ] Enable Redis password authentication

## Common Architecture Issues

### Session Loss After Redis Restart
- **Problem**: All users logged out after Redis restart
- **Cause**: AOF persistence not enabled
- **Solution**: Enable `appendonly yes` for redis_session
- **Reference**: [Redis Architecture Guide](references/redis-architecture.md)

### Cache Evicting Important Data
- **Problem**: Tokens or sessions deleted by cache eviction mechanism
- **Cause**: Storing in cache instance with allkeys-lru
- **Solution**: Separate redis_session (noeviction)
- **Reference**: [Redis Architecture Guide](references/redis-architecture.md)

### Configuration Drift
- **Problem**: Configuration inconsistent across environments
- **Cause**: Manual config management, lack of version control
- **Solution**: Use `.env.example` + Docker Compose
- **Reference**: [Configuration Guide](references/configuration.md)

### Memory Exhaustion
- **Problem**: Redis memory exhausted
- **Cause**: No maxmemory set or wrong eviction policy
- **Solution**: Set maxmemory + correct eviction policy
- **Reference**: [Redis Architecture Guide](references/redis-architecture.md)

## Resources

- [Database ID Strategy Guide](references/database-id-strategy.md) - Primary key design (UUIDv7 vs bigint) with security and performance considerations
- [Redis Architecture Guide](references/redis-architecture.md) - Three-instance separation and configuration
- [Configuration Guide](references/configuration.md) - ENV, Docker Secrets, and Settings management

## Related Skills

- **web-security**: Authentication architecture using Redis + PostgreSQL
- **performance-optimization**: Redis caching for performance
- **deployment-practices**: Docker Compose configuration and deployment
