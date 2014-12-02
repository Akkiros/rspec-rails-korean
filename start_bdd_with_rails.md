# Rspec getting started

##### 참고
http://everydayrails.com/2012/03/19/testing-series-rspec-models-factory-girl.html , http://www.relishapp.com/rspec/rspec-rails/v/3-1/docs

### 설치
http://everydayrails.com/2012/03/12/testing-series-rspec-setup.html

### model test

##### create model

```sh
rails g model tester firstname:string, lastname:string
```
then

```sh
      invoke  active_record
      create    db/migrate/20141124065650_create_testers.rb
      create    app/models/tester.rb
      invoke    rspec
      create      spec/models/tester_spec.rb
      invoke      factory_girl
      create        spec/factories/testers.rb
```

run migration

```sh
rake db:migrate
```

##### 테스트 작성

``spec/factories/tester.rb``에 테스트 데이터를 만든다.

```sh
FactoryGirl.define do
  factory :tester do |f|
    f.firstname "Wonjae"
    f.lastname "Kim"
  end
end
```

``spec/models/tester_spec.rb``에는 테스트를 작성한다.

```sh
require 'rails_helper'
require 'spec_helper'

RSpec.describe Tester, :type => :model do
  it 'is invalid?' do
    expect(FactoryGirl.create(:tester)).to be_valid
  end
```

is invalid? 테스트는 위의 ``spec/factories/tester.rb``에서 만든 tester를 create하고 그게 valid인지 테스트를 하는것이다.

``rspec spec/models/tester_spec.rb``를 해서 valid가 맞다면 테스트를 잘 통과할테고 그게 아니면 Failures가 나올 것이다.

##### model validates test

이번엔 모델에 validates를 추가하고 그것을 테스트 해보자.

``app/models/tester.rb``

```sh
class Tester < ActiveRecord::Base
  validates :firstname, :lastname, presence: true
end
```

``spec/models/tester_spec.rb``

```sh
it 'is invalid without firstname' do
  expect(FactoryGirl.build(:tester, firstname: nil)).to be_valid
end
```

위는 firstname이 nil인 객체가 유효할 것이라는 것에 대해 테스트를 하고 있다. 앞서 firstname과 lastname이 존재해야 한다고 validates를 걸어뒀기 때문에 이 테스트는 실패할 것이다. 테스트를 통과하려면

```sh
it 'is invalid without firstname' do
  expect(FactoryGirl.build(:tester, firstname: nil)).not_to be_valid
end
```

처럼 not_to를 이용하여 그 반대를 테스트할 수 있다.

마찬가지로 uniqueness도 테스트할 수 있다.

```sh
rails g migration add_phone_to_tester
```

``db/migrate/20141124084250_add_phone_to_tester.rb``

```sh
class AddPhoneToTester < ActiveRecord::Migration
  def change
    add_column :testers, :phone, :string
  end
end
```

```sh
rake db:migrate
```

``app/models/tester.rb``

```sh
class Tester < ActiveRecord::Base
  validates :phone, uniqueness: true
end
```

```sh
it 'is not allow duplicate phone number' do
  FactoryGirl.create(:tester, phone: "010-1234-4321")
  expect(FactoryGirl.create(:tester. phone: "010-1234-4321")).to be_valid
end
```

```sh
rspec spec/models/tester_spec.rb
```

result
```sh
Failures:

  1) Tester is not allow duplicate phone
     Failure/Error: expect(FactoryGirl.build(:tester, phone: "010-1234-4321")).to be_valid
       expected #<Tester id: nil, firstname: "John", lastname: "Doe", created_at: nil, updated_at: nil, phone: "010-1234-4321"> to be valid, but got errors: Phone has already been taken
     # ./spec/models/tester_spec.rb:13:in `block (2 levels) in <top (required)>'

Finished in 0.32278 seconds (files took 1.47 seconds to load)
3 examples, 1 failure
```

연락처가 유일하지 않기 때문에 테스트는 실패한다.

##### create()와 build()의 차이점

 - create()
 > create()는 객체를 생성한 후 저장한다. new를 하고 save한것과 같다.
 - build()
 > build()는 객체를 생성한 후 저장하지 않는다. build()로 만든후에 save를 해줘야 저장이 된다. new와 같다.

##### model instance method test

``app/models/tester.rb``에 인스턴스 메소드를 추가해보자.

```sh
class Tester < ActiveRecord::Base
  def name
    [firstname, lastname].join(" ")
  end
end
```

name이라는 인스턴스 메소드는 객체의 firstname과 lastname을 받아 그 사이에 공백을 넣은 후 리턴을 해주는 기능을 가지고 있다.

과연 기댓값이 맞는지 테스트를 작성해보자.

``spec/models/tester_spec.rb``

```sh
it 'return full name as a string' do
  test = FactoryGirl.create(:tester, phone: "010-1234-4321")
  expect(test.name).to eq("Wonjae kim")
end
```

```sh
rspec spec/models/tester_spec.rb
```

우리의 기댓값인 "Wonjae Kim"과 일치를 하기 때문에 아무런 문제가 발생하지 않는다.

