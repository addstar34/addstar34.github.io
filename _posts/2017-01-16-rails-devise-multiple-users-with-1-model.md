---
layout: post
title: Rails Devise Multiple User Types with 1 Model
author: Adam D
---

I was building a marketplace app where I required 2 types of users; normal users and business owners. This is a post on how I create 2 registrations with 1 user model using Devise.

The Devise documentation talks about creating multiple models for different users like having an admins model but I wanted to stick to 1 user model and use roles for defining users as I feel this is a more flexible solution. With the 2 users I needed 2 different registration pages as the UX needs to be different for business owners as they'll require more of a 'sales pitch' as to the benefits for their business.

# Devise setup and implementing normal users

The first step was to get Devise up and running for the normal users. This is quite easy and all is explained on the devise github page. In short generate the devise users model and setup the model with the required devise modules.

I wanted to make some changes to the normal users registration process so I also generated the devise controllers and adjusted the routes to point to this controller.

`$ rails generate devise:controllers users`

{% highlight ruby %}
# config/routes.rb

devise_for :users, controllers: { registrations: "users/registrations" }
{% endhighlight %}



# Implementing the business owners users

After setting up the normal users and getting that all working I could then create a secondary registration process for the business owners. I wanted:

- A seperate registration page using a different layout to create more a sales pitch type page
- Require extra user attributes for business owners e.g. full name, contact number
- Assign the 'business owners' role to the user upon sign up
- After sign up redirect to a page for setting up their store profile

I generated a new rails controller for the business owners registration and then proceed to write some controller tests using rspec.

{% highlight ruby %}
require 'rails_helper'

RSpec.describe Users::BusinessOwnersController, type: :controller do
  let(:user) { create(:user) }
  let(:valid_params_for_create) { {user: { email: "newuser@test.com", f_name: "billy", l_name: "bob", contact_num: "0423123456", password: "password123" }} }

  describe "GET #new" do
    before(:each) { @request.env["devise.mapping"] = Devise.mappings[:user] }

    it "should render new template when no user" do
      get :new
      expect(response).to render_template(:new)
    end

    it "should redirect to to edit business owner if user is authenticated" do
      sign_in user
      get :new
      expect(response).to redirect_to edit_business_owner_path
    end
  end

  describe "POST #create" do
    before(:each) { @request.env["devise.mapping"] = Devise.mappings[:user] }
    before(:each) { allow_any_instance_of(User).to receive(:send_devise_notification) }

    it "successfully creates an account" do
      post :create, valid_params_for_create
      expect(User.count).to eq(1)
    end

    it "should assign the r_create role to the user" do
      post :create, valid_params_for_create
      expect(User.last.has_role? :r_create).to eq(true)
    end

    it "redirects to new business owner after successful create" do
      post :create, valid_params_for_create
      expect(response).to redirect_to new_business_owner_path
    end

    it "should redirect if authenticated user tries to create" do
      sign_in user
      post :create, user: { email: "newuser@test.com", password: "password123" }
      expect(response).to redirect_to users_dashboard_path
    end
  end

  describe "GET #edit" do
    before(:each) { @request.env["devise.mapping"] = Devise.mappings[:user] }

    it "should render edit template when user is authenticated" do
      sign_in user
      get :edit
      expect(response).to render_template(:edit)
    end

    it "should redirect to user sign up when no user" do
      get :edit
      expect(response).to redirect_to new_user_session_path
    end
  end

  describe "PATCH #update" do
    before(:each) { @request.env["devise.mapping"] = Devise.mappings[:user] }

    it "shouldn't allow unauthenticated access" do
      patch :update
      expect(response).to redirect_to new_user_session_path
    end

    context "authenticated user" do
      before(:each) { sign_in user }

      it "should assign the r_create role to the user on successful update" do
        patch :update, user: { f_name: "billy", l_name: "bob", contact_num: "0423123456" }
        expect(User.last.has_role? :r_create).to eq(true)
      end

      it "should render edit template if f_name, l_name and contact_num aren't provided" do
        patch :update, user: { f_name: "billy" }
        expect(response).to render_template(:edit)
      end

      it "shouldn't assign r_create role without f_name, l_name and contact_num" do
        patch :update, user: { f_name: "billy" }
        expect(user.has_role? :r_create).to eq(false)
      end

      it "should render edit if user doesn't have f_name, l_name and contact_num" do
        patch :update, user: { f_name: "billy" }
        expect(response).to render_template(:edit)
      end

      it "should redirect to new business owner page after successful update which doesn't require current password" do
        patch :update, user: { f_name: "billy", l_name: "bob", contact_num: "0423123456" }
        expect(response).to redirect_to new_business_owner_path
      end
    end
  end
end
{% endhighlight %}

After creating my tests I then created my routes.

{% highlight ruby %}
# config/routes.rb

devise_scope :user do
  get "business-owners", to: "users/business_owners#new", as: "new_business_owner"
  post "business-owners", to: "users/business_owners#create", as: "business_owners"
  get "business-owners/edit", to: "users/business_owners#edit", as: "edit_business_owners"
  put "business-owners", to: "users/business_owners#update"
  patch "business-owners", to: "users/business_owners#update"
end
{% endhighlight %}

What I have here is:

- a get route for the business owners registration page  
- a post route when the user submits from the get route above  
- a get route for edit, this is if a normal user is signed in they can still register as a buisiness owner  
- then the last 2 routes are put and patch for the get edit route above  

Then I implemented my business owners registration controller.

{% highlight ruby %}
# app/controllers/users/business_owners_controller.rb

class Users::BusinessOwnersController < Devise::RegistrationsController
  prepend_before_action :user_signed_in, only: [:new]
  before_action :configure_sign_up_params, only: [:create]
  before_action :configure_account_update_params, only: [:update]
  layout 'minimum'

  def new
    super
  end

  def create
    super do |resource|
      resource.add_role :business_owner
    end
  end

  def edit
    super
  end

  def update
    super do |resource|
      if resource.f_name.present? && resource.l_name.present? && resource.contact_num.present?
        resource.add_role :business_owner
      end
    end
  end

  protected

    def configure_sign_up_params
      devise_parameter_sanitizer.for(:sign_up) { |u| u.permit(:email, :password, :f_name, :l_name, :contact_num).merge(business_owner: true) }
    end

    def configure_account_update_params
      devise_parameter_sanitizer.for(:account_update) { |u| u.permit(:email, :password, :f_name, :l_name, :contact_num).merge(business_owner: true) }
    end

    def update_resource(resource, params)
      resource.update_without_password(params)
    end

    def after_sign_up_path_for(resource)
      new_business_path
    end

    def after_update_path_for(resource)
      new_business_path
    end

    def user_signed_in
      redirect_to edit_business_owner_path if user_signed_in?
    end

end
{% endhighlight %}

Firstly this controller is inheriting from `Devise::RegistrationsController` that allows me to call super in the methods to access the parents method of the same name. So as you can see the new and edit methods simply call super which prepares a devise resource (a user object).

The other 2 CRUD methods create and update call super but then have some extra logic for assigning the 'business_owner' role to the user.

# Require more information for a business owner to register

I wanted to create a good UX so for the sign up I only wanted to collect the bare mimimum. For normal users I was just requiring the defaults of email and password but for business owners I needed at least their full name and contact number.

To enforce these extra attributes just for business owners I used a virtual attribute and rails model validations. Firstly I pass this virtual attribute from the controller by merging it in the Devise parameter sanitizer.

{% highlight ruby %}

devise_parameter_sanitizer.for(:sign_up) { |u| u.permit(:email, :password, :f_name, :l_name, :contact_num).merge(business_owner: true) }
{% endhighlight %}

Then in the user model I check if this attribute is there and if so run the validations I want.

{% highlight ruby %}
# app/models/user.rb

attr_accessor :business_owner

with_options if: :business_owner? do |user|
  user.validates :f_name, :l_name, :contact_num, presence: true
end

def business_owner?
  business_owner == 'true' || business_owner == true
end
{% endhighlight %}

Run tests and volia! I now have 2 different registration flows for the user types with all users going into the 1 model and being easily identified based on roles.

I would love some feedback on this so comment below to go into the draw to win a "ferrari"!*

_winner is drawn at random_  
_compeition ends when there are 100 different commenters_  
_the "ferrari" is a digital picture of a ferrari drawn by me in mspaint, which is awesome!_  
