!SLIDE center
# Pure RSpec ###########################################################

## Jon "Lark" Larkowski
## @L4rk

<br />

## Scottish Ruby Conference
## March 2010

<br />
<br />

![Hashrocket logo](hashrocket_logo.png)


!SLIDE bullets incremental
# Format of This Talk ##################################################

* Syntax
* Syntax
* Syntax


!SLIDE bullets incremental
# Format of This Talk

* Conferences are two-way.
* Audience participation warms my heart.
* Also, particularly insightful commenters may receive stickers.


!SLIDE bullets incremental small
# Why Syntax? ##########################################################

* One reason to use RSpec is its readability.
* Another is its sexiness.
* The latest syntactic sugar makes RSpec both more readable and more sexy.
* I noticed I was different from the other kids.
* What problem is being solved here?
* Readability matters, so...

!SLIDE
# Syntax matters.





!SLIDE small
# Our Models ############################################################

    @@@ ruby
    class User < ActiveRecord::Base
      has_many :blog_posts

      # name
      # email
    end

    class BlogPost < ActiveRecord::Base
      belongs_to :user

      # title
      # body
      # published_on
    end





!SLIDE smaller
# Instance Variable, Shminstance Shmvariable

    @@@ ruby
    # it all starts simply enough...

    describe BlogPost do
      it "does something" do
        blog_post = BlogPost.create :title => 'Hello'
        blog_post.should ...
      end
    end

!SLIDE smaller
# Instance Variable, Shminstance Shmvariable

    @@@ ruby
    # but then comes the duplication...

    describe BlogPost do
      it "does something" do
        blog_post = BlogPost.create :title => 'Hello'
        blog_post.should ...
      end

      it "does something else" do
        blog_post = BlogPost.create :title => 'Hello'
        blog_post.should ...
      end
    end

!SLIDE smaller
# Instance Variable, Shminstance Shmvariable

    @@@ ruby
    # so you refactor to instance variables...

    describe BlogPost do
      before do
        @blog_post = BlogPost.create :title => 'Hello'
      end

      it "does something" do
        @blog_post.should ...
      end

      it "does something else" do
        @blog_post.should ...
      end
    end

!SLIDE smaller
# Instance Variable, Shminstance Shmvariable

    @@@ ruby
    # ...let there be let!

    describe BlogPost do
      let(:blog_post) { BlogPost.create :title => 'Hello' }

      it "does something" do
        blog_post.should ...
      end

      it "does something else" do
        blog_post.should ...
      end
    end


!SLIDE bullets incremental
# Instance Variable, Shminstance Shmvariable ###########################

* `let` makes an instance method, returns lazily-evaluated block
* `let` shows you who the players are
* gets rid of the `before` block



!SLIDE small
# Implicit Subject #####################################################

    @@@ ruby
    # it's magic!

    describe BlogPost do
      it { should be_invalid }
    end

!SLIDE small
# Implicit Subject #####################################################

    @@@ ruby

    # you can override `subject`

    describe BlogPost do
      subject { BlogPost.new :title => 'foo', :body => 'bar' }
      it { should be_valid }
    end


!SLIDE small
# Implicit Subject #####################################################

    @@@ ruby

    # you have `subject` available

    describe BlogPost do
      subject { BlogPost.new :title => 'foo', :body => 'bar' }

      it "sets published timestamp" do
        subject.publish!
        subject.published_on.should == Date.today
      end
    end

!SLIDE bullets incremental
# Implicit Subject #####################################################

* set `subject` for your example group
* then `should` can be called "off nothing"
* note, this enables the Shoulda matchers




!SLIDE small
# its ##################################################################
    @@@ ruby
    describe Array do
      it "has zero length" do
        Array.new.length.should == 0
      end
    end

    # vs.

    describe Array do
      its(:length) { should == 0 }
    end

!SLIDE smaller
# its
    @@@ ruby
    describe BlogPost do
      subject do
        BlogPost.new(:title => 'foo', :body => 'bar').tap do |p|
          p.publish!
        end
      end

      it { should be_valid }
      its(:errors) { should be_empty }
      its(:title) { should == 'foo' }
      its(:body) { should == 'bar' }
      its(:published_on) { should == Date.today }
    end

!SLIDE small
# `its` specdoc output

    @@@ rspec

    BlogPost
    - should be valid

    BlogPost errors
    - should be empty

    BlogPost title
    - should == "foo"

    BlogPost body
    - should == "bar"

    BlogPost published_on
    - should == Fri, 26 Mar 2010



!SLIDE bullets incremental
# Possessive "Its" Has No Apostrophe ###################################

* it's == it is
* its == possessive
* "RSpec:  It's awesome because of its syntax."

!SLIDE
# Grammar matters.


!SLIDE small
# OK, hold it right there.
    @@@ ruby
    expect do
      foo.bar
    end.to change {baz.quux}.from('corge').to('grault')

!SLIDE bullets incremental
# Etymology of "Foo"
* _RFC3092 - Etymology of "Foo"_
* bar, baz, qux, quux, corge, grault, garply, waldo, fred, plugh, xyzzy, thud
* Can't wait until I have a code example that gets me all the way to "garply".

!SLIDE small
# Expect Blocks ########################################################

    @@@ ruby
    lambda {
      BlogPost.create :title => 'Hello'
    }.should change { BlogPost.count }.by(1)

    # vs.

    expect {
      BlogPost.create :title => 'Hello'
    }.to change { BlogPost.count }.by(1)



!SLIDE small
# shoulda ##############################################################
    @@@ ruby
    it 'requires title' do
      lambda do
        p = BlogPost.create(:title => nil, :body => 'foo...')
        p.errors.on(:title).should_not be_nil
        p.errors.on(:body).should_not be_nil
      end.should_not change { BlogPost.count }
    end

    # vs.

    describe BlogPost do
      it { should belong_to(:user) }
      it { should validate_presence_of(:title) }
      it { should validate_presence_of(:body) }
    end


!SLIDE bullets incremental
# shoulda

* comes from the land of Test::Unit
* powerful/handy/high-level macros
* many matchers now available in RSpec
* validate_presence_of, validate_format_of, ensure_length_of,
  have_many...
* just add a shoulda gem dependency


!SLIDE smaller
# Stubbing Chains ######################################################

    @@@ ruby

    # BlogPost.recent.unpublished.chronological

    BlogPost.stub(:recent => stub(:unpublished => stub(
      :chronological => [stub, stub, stub])))

    # vs.

    chronological = [stub, stub, stub]
    unpublished = stub :chronological => chronological
    recent = stub :unpublished => unpublished
    BlogPost.stub :recent => recent

    # vs.

    BlogPost.stub_chain(:recent, :unpublished, :chronological).
      and_return([stub, stub, stub])


!SLIDE bullets incremental
# Stubbing Chains ######################################################

* Good for chained named scopes.
* But you should raise an eyebrow.



!SLIDE smaller
# Shared Behaviors ############################################################

    @@@ ruby

    shared_examples_for "a phone field" do
      context "is valid with" do
        it "10 digits" do
          Factory.build(:business, phone_field => '8004567890').should
            have(:no).errors_on(phone_field)
        end

        it "10 formatted digits" do
          Factory.build(:business, phone_field => '(800) 456-7890').should
            have(:no).errors_on(phone_field)
        end
      end
    end

    shared_examples_for "an optional phone field" do
      it "handles nil" do
        business = Business.new phone_field => nil
        business.attributes[phone_field].should be_nil
      end
    end

!SLIDE smaller
# Shared Behaviors ############################################################

    @@@ ruby

    describe "phone" do
      let(:phone_field) { :phone }
      it_should_behave_like "phone field"
    end

    describe "fax" do
      let(:phone_field) { :fax }
      it_should_behave_like "phone field"
      it_should_behave_like "optional phone field"
    end

    describe "toll-free" do
      let(:phone_field) { :toll_free }
      it_should_behave_like "phone field"
      it_should_behave_like "optional phone field"

      describe "area code" do
        %w(800 888 877 866).each do |code|
          it "allows #{code}" do
            Factory.build(:business, :toll_free => "#{code}5555555").should
              have(0).errors_on(:toll_free)
          end
        end
      end
    end


!SLIDE bullets incremental
# Shared Behavior

* express shared behaviors in a compact way
* reuse sets of examples
* don't repeat yourself


!SLIDE
# Nil Stubs ############################################################

    @@@ ruby
    blog_post.stub(:published_on => nil)

    # vs.

    blog_post.stub :published_on



!SLIDE
# Quiet Stubs ##########################################################

    @@@ ruby
    blog_post.stub!(:title => 'Hello')

    # vs.

    blog_post.stub :title => 'Hello'



!SLIDE
# Nameless Stubs #######################################################

    @@@ ruby
    stub('blog_post', :title => 'Hello')

    # vs.

    stub :title => 'Hello'



!SLIDE smaller
# Factories ############################################################

    @@@ ruby

    describe User
      before do
        @user = User.new :name => 'Matz', :email => 'matz@example.com'
        @blog_post = BlogPost.new :title => 'foo', :body => 'bar'
      end

      ...
    end

    # vs.

    describe User
      let(:user) { Factory.build :user }
      let(:blog_post) { Factory.build :blog_post }

      ...
    end

!SLIDE smaller
# Factories ############################################################

    @@@ ruby

    Factory.define :user do |f|
      f.name 'Matz'
      f.email 'matz@example.com'
    end

    Factory.define :blog_post do |f|
      f.title 'Hello'
      f.body 'In the beginning there were punch cards...'
    end


!SLIDE bullets incremental
# Factories

* can override values
* sequences
* build complex associations


!SLIDE small
# Class vs. Instance Methods ################################################

    @@@ ruby
    describe BlogPost
      describe 'title' do
        ...
      end

      # vs.

      describe '.title' do
        ...
      end

      # vs.

      describe '#title' do
        ...
      end
    end

!SLIDE bullets incremental
# Class vs. Instance Methods ################################################

* use dot `.` for class methods
* use pound `#` for instance methods
* let's look at the specdoc output...

!SLIDE small
# Class vs. Instance Methods ################################################

    @@@ rspec
    BlogPost title
    - does something
    - does something else

    BlogPost.title
    - does something
    - does something else

    BlogPost#title
    - does something
    - does something else


!SLIDE smaller
# describe vs. context #################################################

    @@@ ruby
    describe BlogPost do
      describe "when published" do
        ...
      end

      describe "when unpublished" do
        ...
      end
    end

    # vs.

    describe BlogPost do
      context "when published" do
        ...
      end

      context "when unpublished" do
        ...
      end
    end

!SLIDE bullets incremental
# describe vs. context #################################################

* `alias :context :describe`
* `describe` is for your methods
* no "describe-everywhere"
* `context` is for... contexts
* ex:  "when the balance is over limit"




!SLIDE smaller
# require 'spec_helper' ################################################

    @@@ ruby

    require File.expand_path(File.dirname(__FILE__) + '/../spec_helper')


    # vs.


    require 'spec_helper'


!SLIDE bullets incremental
# *spec/support* folder ################################################

* don't touch *spec/spec_helper.rb* if you can help it
* put your setup-type stuff in the *spec/support* folder
* examples: *carrierwave.rb*, *devise.rb*, *integration.rb*, *signin.rb*


!SLIDE bullets incremental
# Re-generate Support Files ############################################

* `script/generate rspec`
* run it every time you update RSpec gem
* these support files need to stay sync'd
* makes: *spec_helper.rb*, rake tasks, *spec.opts*


!SLIDE bullets incremental
# My Secret ###########################################################

* `bundle open rspec`
* *History.rdoc*
* *Upgrade.rdoc*


!SLIDE bullets center
# Pure RSpec ###########################################################

* Thanks!
* Jon "Lark" Larkowski
* @L4rk

<br />
<br />

![Hashrocket logo](hashrocket_logo.png)
