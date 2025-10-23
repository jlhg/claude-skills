# Partial doubles

When the `verify_partial_doubles` configuration option is set, the same argument and
  method existence checks that are performed for [`object_double`](./object-doubles) are also performed on
  [partial doubles](../basics/partial-test-doubles). You should set this unless you have a good reason not to. It defaults to off
  only for backwards compatibility.

## Doubling an existing object

_Given_ a file named "spec/user_spec.rb" with:

```ruby
class User
  def save; false; end
end

def save_user(user)
  "saved!" if user.save
end

RSpec.configure do |config|
  config.mock_with :rspec do |mocks|
    mocks.verify_partial_doubles = true
  end
end

RSpec.describe '#save_user' do
  it 'renders message on success' do
    user = User.new
    expect(user).to receive(:wave).and_return(true) # Typo in name
    expect(save_user(user)).to eq("saved!")
  end
end
```

_When_ I run `rspec spec/user_spec.rb`

_Then_ the output should contain "1 example, 1 failure".
