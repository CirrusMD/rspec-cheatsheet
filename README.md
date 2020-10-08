# rspec-cheatsheet

Documenting some of the intricacies of the RSpec Testing Framework

## Topics

- [Pending Specs](#pending)

#### Pending

There are three ways to mark a spec as pending:

1. `pending`
1. `skip`
1. `xit`

**pending**

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

**skip**

Skip is used in the same exact way, but it will not attempt to run the code inside
the block and will not fail once the spec produces no failures.

**xit**

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
