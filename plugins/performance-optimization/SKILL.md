---
name: Optimizing Application Performance
description: Web application performance optimization techniques covering memory management, jemalloc configuration, and memory leak detection. Use when optimizing high-traffic Rails applications, reducing memory usage, performance tuning, or when user mentions memory optimization, jemalloc, memory leak, Ruby, Rails, performance, profiling, derailed_benchmarks, or memory_profiler.
---

# Performance Optimization

## Overview

Best practices focused on web application performance optimization, particularly memory management and performance profiling tools. These techniques are primarily for Ruby on Rails applications, but many concepts apply to other web frameworks.

## Memory Optimization

Memory optimization is critical for high-traffic applications. See [Memory Optimization Guide](references/memory-optimization.md) for:

### jemalloc

jemalloc is a high-performance memory allocator that can significantly reduce Ruby application memory usage.

- **Why Use jemalloc**
  - Reduces memory fragmentation
  - Better multi-threaded performance
  - Lowers overall memory usage (20-30% reduction)
  - Improves garbage collection efficiency

- **Docker Integration**
  - Dockerfile configuration
  - Environment variable setup (`LD_PRELOAD`, `MALLOC_CONF`)
  - Optimal tuning parameters

- **Performance Measurement**
  - Before/after memory usage comparison
  - Monitor RSS, heap size
  - Evaluate performance impact

### Memory Leak Detection

Finding and fixing memory leak issues.

- **Symptom Identification**
  - Memory continuously growing
  - Returns to normal after restart
  - Memory not reclaimed by GC

- **Detection Tools**
  - `derailed_benchmarks` - Measure gem memory usage
  - `memory_profiler` - Analyze memory allocation
  - Production monitoring metrics

- **Common Memory Leak Sources**
  - Global variable accumulation
  - Class variables storing request data
  - Callback closures retaining references
  - Third-party gem issues

- **Fix Strategies**
  - Use `ObjectSpace` to track objects
  - Analyze gem memory usage
  - Refactor problematic code
  - Upgrade or replace problematic gems

### Memory Profiling

Using professional tools to analyze memory usage.

- **derailed_benchmarks**
  - Measure memory footprint of each gem
  - Identify memory-intensive dependencies
  - Evaluate upgrade or removal impact

- **memory_profiler**
  - Track memory allocation in specific code
  - Find allocation hotspots
  - Optimize object creation

- **Production Monitoring**
  - Track RSS, heap size, GC statistics
  - Set memory alerts
  - Regular trend analysis

## Optimization Strategies

### String Memory Optimization

String memory optimization is crucial for Ruby applications.

- **Frozen String Literals**
  - Enable `frozen_string_literal: true`
  - Reduce string object allocation
  - See: [ruby-development skill - Frozen String Literal Guide]

### Rate Limiting for Performance

Rate limiting is not just a security measure, but also a performance protection mechanism.

- **Preventing Resource Exhaustion**
  - Limit frequency of expensive operations
  - Protect database connection pool
  - See: [web-security skill - Rate Limiting Guide]

### Caching Strategies

Caching is core to performance optimization.

- **Redis Caching Architecture**
  - Separate cache, session, cable instances
  - Appropriate eviction policy
  - See: [backend-architecture skill - Redis Architecture Guide]

## Performance Checklist

Checklist for optimizing application performance:

### Memory Management
- [ ] Enable jemalloc
- [ ] Configure `MALLOC_CONF` environment variable
- [ ] Monitor production memory usage
- [ ] Regularly perform memory profiling
- [ ] Check for memory leak symptoms

### Code Optimization
- [ ] Enable `frozen_string_literal: true`
- [ ] Analyze gems with `derailed_benchmarks`
- [ ] Optimize N+1 queries
- [ ] Use appropriate database indexes
- [ ] Implement caching strategies

### Production Monitoring
- [ ] Set memory alerts (RSS, heap size)
- [ ] Track GC statistics
- [ ] Monitor response time
- [ ] Analyze slow query logs
- [ ] Regularly review performance trends

## Common Performance Issues

### Memory Bloat
- **Problem**: Memory usage grows over time
- **Diagnosis**: Use `derailed_benchmarks`, `memory_profiler`
- **Solution**: Enable jemalloc, fix memory leaks
- **Reference**: [Memory Optimization Guide](references/memory-optimization.md)

### Slow Response Times
- **Problem**: API response time too long
- **Diagnosis**: APM tools, database slow query logs
- **Solution**: Optimize queries, add caching, use CDN
- **Related**: Rate limiting can protect system

### Resource Exhaustion
- **Problem**: Database connections, memory exhausted
- **Diagnosis**: Monitor resource utilization
- **Solution**: Rate limiting, connection pool tuning, vertical/horizontal scaling
- **Reference**: [web-security skill - Rate Limiting Guide]

## Profiling Tools

### Ruby Profiling
- `derailed_benchmarks` - Gem memory analysis
- `memory_profiler` - Memory allocation tracking
- `rack-mini-profiler` - Real-time performance analysis
- `stackprof` - CPU profiling

### System Monitoring
- `htop` / `top` - System resource monitoring
- `vmstat` - Virtual memory statistics
- `iostat` - I/O statistics

### Production Monitoring
- Datadog / New Relic - APM tools
- Prometheus + Grafana - Metrics monitoring
- ELK Stack - Log analysis

## Resources

- [Memory Optimization Guide](references/memory-optimization.md) - jemalloc setup and memory leak detection

## Related Skills

- **ruby-development**: Frozen string literal optimization
- **web-security**: Rate limiting for performance protection
- **backend-architecture**: Redis caching architecture
- **deployment-practices**: Zero-downtime deployment performance considerations
