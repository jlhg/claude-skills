# Configurable colors

RSpec allows you to configure the terminal colors used in the text formatters.

  * `failure_color`: Color used when tests fail (default: `:red`)
  * `success_color`: Color used when tests pass (default: `:green`)
  * `pending_color`: Color used when tests are pending (default: `:yellow`)
  * `fixed_color`: Color used when a pending block inside an example passes, but
    was expected to fail (default: `:blue`)
  * `detail_color`: Color used for miscellaneous test details (default: `:cyan`)

  Colors are specified as symbols. Options are `:black`, `:red`, `:green`,
  `:yellow`, `:blue`, `:magenta`, `:cyan`, `:white`, `:bold_black`, `:bold_red`,
  `:bold_green`, `:bold_yellow`, `:bold_blue`, `:bold_magenta`, `:bold_cyan`,
  and `:bold_white`,

## Customizing the failure color

_Given_ a file named "custom_failure_color_spec.rb" with:

```ruby
RSpec.configure do |config|
  config.failure_color = :magenta
  config.color_mode = :on
end

RSpec.describe "failure" do
  it "fails and uses the custom color" do
    expect(2).to eq(4)
  end
end
```

_When_ I run `rspec custom_failure_color_spec.rb --format progress`

_Then_ the failing example is printed in magenta.

## Customizing the failure color with a custom console code

_Given_ a file named "custom_failure_color_spec.rb" with:

```ruby
RSpec.configure do |config|
  config.failure_color = "1;32"
  config.color_mode = :on
end

RSpec.describe "failure" do
  it "fails and uses the custom color" do
    expect(2).to eq(4)
  end
end
```

_When_ I run `rspec custom_failure_color_spec.rb --format progress`

_Then_ the failing example is printed wrapped in "1;32".
