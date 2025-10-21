# RSpec Best Practices

Based on [Better Specs](https://www.betterspecs.org/)

## Table of Contents

- [Test Organization & Description](#test-organization--description)
- [Syntax & Conventions](#syntax--conventions)
- [Testing Strategy](#testing-strategy)
- [Data Management](#data-management)
- [Tools & Performance](#tools--performance)
- [Configuration](#configuration)

## Test Organization & Description

### Method Naming in Describe Blocks

Use `.` (or `::`) for class methods, `#` for instance methods:

```ruby
# Good
describe '.authenticate' do
  # class method tests
end

describe '#admin?' do
  # instance method tests
end
```

### Context Usage

Use contexts with 'when', 'with', or 'without' prefixes to organize related tests:

```ruby
# Good
context 'when logged in' do
  it { is_expected.to respond_with 200 }
end

context 'when logged out' do
  it { is_expected.to respond_with 401 }
end

context 'with valid params' do
  # tests
end

context 'without required fields' do
  # tests
end
```

### Description Guidelines

- Keep descriptions short (< 40 characters)
- Use context to add detail rather than long descriptions
- Use present tense, not "should"

```ruby
# Bad
it 'should not change timings' do

# Good
it 'does not change timings' do
```

## Syntax & Conventions

### Expect Syntax

Use `expect` syntax, not `should`:

```ruby
# Good
expect(response).to respond_with_content_type(:json)
expect(user.name).to eq('John')

# Bad
response.should respond_with_content_type(:json)
```

### One-liner Syntax

Use `is_expected.to` for one-liners with implicit subject:

```ruby
# Good
context 'when not valid' do
  it { is_expected.to respond_with 422 }
end

describe '#published?' do
  subject { post.published? }
  
  context 'when published_at is set' do
    let(:post) { create(:post, published_at: Time.current) }
    it { is_expected.to be true }
  end
end
```

### Subject Usage

Use `subject` to DRY up related tests:

```ruby
# Good
subject { assigns('message') }
it { is_expected.to match /it was born in Billville/ }

# Named subject
subject(:hero) { Hero.first }
it "carries a sword" do
  expect(hero.equipment).to include "sword"
end
```

### Let vs Before

Use `let` (lazy load) instead of `before` with instance variables:

```ruby
# Good
describe '#type_id' do
  let(:resource) { FactoryBot.create :device }
  let(:type)     { Type.find resource.type_id }

  it 'sets the type_id field' do
    expect(resource.type_id).to eq(type.id)
  end
end

# Bad
describe '#type_id' do
  before do
    @resource = FactoryBot.create :device
    @type = Type.find @resource.type_id
  end

  it 'sets the type_id field' do
    expect(@resource.type_id).to eq(@type.id)
  end
end
```

### Let vs Let!

- Use `let` for lazy loading (loaded only when referenced)
- Use `let!` for eager loading (e.g., database population for queries/scopes)

```ruby
# Lazy loading - user is only created when referenced in test
let(:user) { create(:user) }

# Eager loading - users are created before each test
let!(:users) { create_list(:user, 3) }
```

### Context Variable Redefinition

Context uses `let` to redefine variables (not `let!`). Outer `before` blocks automatically use new definitions:

```ruby
describe '#approve' do
  let(:user) { create(:user) }
  before { service.approve(user) }

  context 'when user is admin' do
    let(:user) { create(:user, :admin) }  # Redefines user
    # before block uses admin user automatically
    
    it 'approves immediately' do
      expect(approval).to be_approved
    end
  end

  context 'when user is regular' do
    # Uses original user definition
    it 'requires additional approval' do
      expect(approval).to be_pending
    end
  end
end
```

Don't repeat `before` in contexts - reuse outer `before` blocks.

## Testing Strategy

### Single vs Multiple Expectations

**Isolated unit specs:** Use single expectation per test

```ruby
# Good (isolated)
it { is_expected.to respond_with_content_type(:json) }
it { is_expected.to assign_to(:resource) }
```

**Non-isolated tests:** Multiple expectations OK for integration, database, or external service tests to avoid performance hit

```ruby
# Good (not isolated - integration test)
it 'creates a resource' do
  expect(response).to respond_with_content_type(:json)
  expect(response).to assign_to(:resource)
  expect(Resource.count).to eq(1)
end
```

### Test All Cases

Test valid, edge, and invalid cases:

```ruby
# Good
describe '#destroy' do
  context 'when resource is found' do
    it 'responds with 200'
    it 'shows the resource'
  end

  context 'when resource is not found' do
    it 'responds with 404'
  end

  context 'when resource is not owned' do
    it 'responds with 404'
  end
  
  context 'when resource has dependencies' do
    it 'responds with 422'
    it 'returns error message'
  end
end
```

### Mocking Strategy

Test real behavior when possible. Use mocks sparingly:

```ruby
# Use mocks for specific scenarios like error conditions
context "when service is unavailable" do
  before do
    allow(ExternalService).to receive(:call)
      .and_raise(ExternalService::ConnectionError)
  end

  it { is_expected.to respond_with 503 }
end

# Test real behavior for normal cases
context "when service responds successfully" do
  # Use real service call or VCR cassette
  it 'returns parsed data' do
    result = service.fetch_data
    expect(result).to include(:id, :name)
  end
end
```

### Validation Usage

Use `match(...)` and `hash_including(...)` for flexible validations:

```ruby
# Good
expect(response_body).to match(
  id: be_a(Integer),
  name: 'John',
  email: match(/@example\.com$/)
)

expect(user_attributes).to hash_including(
  name: 'John',
  email: 'john@example.com'
)

# Allows additional keys not specified
expect(response).to include('status' => 'success')
```

## Data Management

### Minimal Data Creation

Create only the data you need:

```ruby
# Good
describe ".top" do
  before { FactoryBot.create_list(:user, 3) }
  it { expect(User.top(2)).to have(2).items }
end

# Bad - creates unnecessary data
describe ".top" do
  before { FactoryBot.create_list(:user, 100) }
  it { expect(User.top(2)).to have(2).items }
end
```

### Factories over Fixtures

Use factories (FactoryBot), not fixtures:

```ruby
# Good
user = FactoryBot.create :user
admin = FactoryBot.create :user, :admin

# Can also use short form if configured
user = create :user
admin = create :user, :admin
```

### Shared Examples

Use shared examples to DRY up tests:

```ruby
# Define shared example
RSpec.shared_examples 'a listable resource' do
  it 'returns a list' do
    expect(response).to have_http_status(200)
    expect(json_response).to be_an(Array)
  end
end

# Use shared example
describe 'GET /devices' do
  let!(:resource) { FactoryBot.create :device, created_from: user.id }
  let(:uri)       { '/devices' }

  it_behaves_like 'a listable resource'
  it_behaves_like 'a paginable resource'
  it_behaves_like 'a searchable resource'
  it_behaves_like 'a filterable list'
end
```

## Tools & Performance

### Guard for Automatic Testing

Use Guard for automatic test running during development:

```ruby
bundle exec guard
```

### Preloaders

Use preloaders (Zeus, Spring) for faster Rails test startup.

### HTTP Request Stubbing

Stub HTTP requests with webmock/VCR:

```ruby
# Good
context "with unauthorized access" do
  let(:uri) { 'http://api.example.com/types' }
  before do
    stub_request(:get, uri)
      .to_return(status: 401, body: fixture('401.json'))
  end

  it "gets a not authorized notification" do
    page.driver.get uri
    expect(page).to have_content 'Access denied'
  end
end

# With VCR
it "fetches user data", vcr: true do
  user = ExternalAPI.fetch_user(123)
  expect(user.name).to eq('John')
end
```

### Better Output Formatting

Use formatters like fuubar for better test output:

```ruby
# Gemfile
group :development, :test do
  gem 'fuubar'
end

# .rspec
--format Fuubar
--color
```

## Configuration

### Expect Syntax Only

Configure RSpec to use expect syntax only (recommended for new projects):

```ruby
# spec/spec_helper.rb
RSpec.configure do |config|
  config.expect_with :rspec do |c|
    c.syntax = :expect
  end
end
```

### Useful Configuration Options

```ruby
# spec/spec_helper.rb
RSpec.configure do |config|
  # Use color in output
  config.color = true
  
  # Use documentation format
  config.default_formatter = 'doc'
  
  # Show 10 slowest examples
  config.profile_examples = 10
  
  # Run specs in random order
  config.order = :random
  
  # Fail fast - stop on first failure
  config.fail_fast = true
end
```

## Summary Checklist

- [ ] Use `.` for class methods, `#` for instance methods in describe
- [ ] Use `context` with 'when/with/without' prefixes
- [ ] Write descriptions in present tense without "should"
- [ ] Use `expect` syntax, not `should`
- [ ] Use `let` for lazy loading, `let!` for eager loading
- [ ] Use `is_expected.to` for one-liners
- [ ] Single expectation for unit tests, multiple OK for integration
- [ ] Test valid, edge, and invalid cases
- [ ] Use factories, not fixtures
- [ ] Create minimal test data
- [ ] Use shared examples for DRY tests
- [ ] Stub external HTTP requests
- [ ] Use `match` and `hash_including` for flexible validations
- [ ] Redefine variables with `let` in contexts, not `let!`
- [ ] Reuse outer `before` blocks in nested contexts
