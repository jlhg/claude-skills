# RSpec Additional Tips

Personal supplementary RSpec best practices.

## Avoid Duplicate Before Blocks

### Problem

Nested contexts with duplicate `before` blocks execute operations (HTTP requests, queries) multiple times.

```ruby
# ❌ Bad - request runs twice
describe 'GET /api/users/default' do
  before { get '/api/users/default', params: { token: access_token } }

  context 'when no default exists' do
    before do
      User.update_all(is_default: false)
      get '/api/users/default', params: { token: access_token }  # Duplicate!
    end
  end
end
```

### Solution: Redefine `let` Variables

Outer `before` blocks automatically use redefined variables from nested contexts.

```ruby
# ✅ Good - request runs once
describe 'GET /api/users/default' do
  let(:user) { create(:user, is_default: true) }
  let(:access_token) { user.access_token }

  before { get '/api/users/default', params: { token: access_token } }

  it { expect(response).to have_http_status(:ok) }

  context 'when no default exists' do
    let(:user_without_default) { create(:user, is_default: false) }
    let(:access_token) { user_without_default.access_token }
    # Outer before automatically uses new access_token

    it { expect(response).to have_http_status(:not_found) }
  end
end
```

### Best Practices

✅ Use one outer `before` block
✅ Redefine `let` variables in contexts

❌ Don't duplicate `before` blocks in nested contexts
