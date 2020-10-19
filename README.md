# rspec-cheatsheet

Documenting some of the intricacies of the RSpec Testing Framework

## Topics

- [Pending Specs](#pending)
- [Shared Setup](#shared-setup)

#### Pending

There are three ways to mark a spec as pending:

1. `pending`
1. `skip`
1. `xit`
1. no block

##### pending

Pending will still run the block, but not count failures that occur within the block.
This will output a message that this spec is failing but pending in the terminal.
The nice thing about pending is that when the specs run and does not produce a
failure, it will mark the test as failing. This is helpful when using TDD and using
the specs as an implementation guide.

It accepts an optional string argument to output any reasoning for why the spec
is marked pending.

```ruby
RSpec.describe FakeClass do
  RSpec.describe '#run' do
    it 'runs the code inside the #run method' do
      pending 'Not working yet'
      expect(subject.run).to be_truthy
    end
  end
end
```

##### skip

Skip is used in the same exact way, but it will not attempt to run the code inside
the block and will not fail once the spec produces no failures.

##### xit

This behaves the same skip, but can actually be defined on any higher level
describe block to skip entire suites or logical test chunks.

```ruby
RSpec.describe FakeClass do
  xRSpec.describe '#run' do
  # Will skip all specs inside the '#run' describe block
    xit 'runs the code inside the #run method' do
    # Will skip only this spec
      expect(subject.run).to be_truthy
    end
  end
end
```

##### missing block

If you leave the block off a test, it will automatically mark it as pending.

```ruby
RSpec.describe FakeClass do
  RSpec.describe '#run' do
    it 'is pending unless I pass a block'
  end
end
```

#### Shared Setup

There are several ways to share context within a spec file.

1. `let` constructs
1. helper methods
1. hooks
1. `shared_context`

##### let constructs

You can re-use common objects between specs by specifying them in `let` blocks.
The difference between `let` and `let!` is the code inside `let` is not run until
it is explicitly called upon, whereas `let!` is run prior to the test run.

```ruby
RSpec.describe FakeClass do
  let(:data) { { attr1: 'hello', attr2: 'data' } }

  it 'runs' do
    # data is present because it is created when the "data" method is run the
    # first time.
    expect(data).to be_truthy
  end
end
```

This doesn't work for anything that might query that data outside the code, like
if the code fetches something from the database as part of its logical flow. For
that, you'll need `let!`

```ruby
RSpec.describe FakeClass do
  let!(:data) { FactoryBot.create(:data, attr1: 'hello', attr2: 'data') }

  it 'runs' do
    expect(FakeClass.new.relevant_data).to be_truthy
  end
end
```

Here, we can assume Fake Class is querying data from the db and making it
available in the "relevant_data" attribute. Using a library like FactoryBot
allows us to add data to the database easily. Nothing called the "data" method to
create that data for us in the spec, so we know that `let!` is running before the
spec runs to ensure that data is avilable already.

##### helper methods

You can also move the creation of shared data to a method, it works and looks
similarly to the let constructs.

```ruby
RSpec.describe FakeClass do
  it 'runs' do
    expect(instance_object.relevant_data).to be_truthy
  end

  def instance_object
    FakeClass.new
  end
end
```

##### hooks

Hooks are blocks that run before test suites to allow you to share set up. There
are several scopes that specify which parts of your tests to run before: the
entire suite, before all specs in a file, or even before specific specs in a
context block.

```ruby
RSpec.describe FakeClass do
  let(:data) { FactoryBot.create(:data, attr1: 'hello', attr2: 'data') }

  before do
    class_double = OpenStruct.new(relevant_data: data)
    allow(FakeClass).to receive(:new).and_return(class_double)
  end

  it 'runs' do
    expect(FakeClass.new.relevant_data).to be_truthy
  end
end
```

Here we are using some stubbing to intercept an objects messages. By putting it
in the before hook we can share among any specs at the same level.

```ruby
RSpec.describe FakeClass do
  let(:data) { FactoryBot.create(:data, attr1: 'hello', attr2: 'data') }

  before do
    # run code for all specs in this suite
  end

  it 'runs' do
    expect(FakeClass.new.relevant_data).to be_truthy
  end

  context 'specific context' do
    before do
      # run code for only specs in this context
    end

    it 'runs with context' {}
    it 'fails with context' {}
  end
end
```

##### shared_context

Lets say I had some integral class that was obnoxious to set up, but much of
the applications logic centered around this being setup properly. We can use
something called "shared context" to make it dead simple to retrieve some shared
setup from within any spec anywhere in our tests.

```ruby
RSpec.shared_context 'MainClass Setup' do
  def main
    args = {}
    MainClass.new(**args)
  end
end
```

Now within our spec:

```ruby
RSpec.describe AnotherClass do
  include_context 'MainClass Setup'

  it 'does anything' do
    expect(subject.stuff).to eq main
  end
end
```

These are contrived examples, but hopefully describe the power of shared test setup.
