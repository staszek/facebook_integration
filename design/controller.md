!SLIDE
#Projekt apliakcji#
!SLIDE
#Baza danych#
!SLIDE

	@@@ ruby

		class User < ActiveRecord::Base
			validates_presence_of :email
		end
!SLIDE

	@@@ ruby

		class User < ActiveRecord::Base
			validates_presence_of :email
			validates_presence_of :facebook_id
		end
!SLIDE

	@@@ ruby

		class User < ActiveRecord::Base
			validates_presence_of :email
			validates_presence_of :facebook_id
			validates_presence_of :twitter_id
		end
!SLIDE

	@@@ ruby

		class User < ActiveRecord::Base
			validates_presence_of :email
			validates_presence_of :facebook_id
			validates_presence_of :twitter_id
			validates_presence_of :facebook_token
			validates_presence_of :twitter_token
		end
!SLIDE

	@@@ ruby

		class User < ActiveRecord::Base
			validates_presence_of :email

			has_many :identifications
		end
!SLIDE smaller

	@@@ ruby

	class Identification < ActiveRecord::Base
		PROVIDERS = %w(facebook)

		belongs_to :user

		validates_presence_of :uid
		validates_presence_of :token
		validates_inclusion_of :provider, :in => PROVIDERS

		def from_facebook?
			provider == 'facebook'
		end
	end
	
!SLIDE bullets incremental smaller
# Przypadki użycia #
  * Podłączenie konta do zalogowanego użytkownika
  * Logowanie przez facebooka
  * Rejestracja przez facebooka

!SLIDE
# Kontroler #
!SLIDE
# UWAGA! #
!SLIDE smaller

	@@@ ruby
	
	def create
		omniauth = request.env["omniauth.auth"]
		identification = Identification.find_by_provider_and_uid(omniauth['provider'], omniauth['uid'])
		if identification
			flash[:notice] = "Signed in successfully."
			sign_in_and_redirect(:user, identification.user)
		elsif current_user
			current_user.identifications.create!(:provider => omniauth['provider'], :uid => omniauth['uid'])
			flash[:notice] = "Identification successful."
			redirect_to identifications_url
		else
			user = User.new
			user.apply_omniauth(omniauth)
			if user.save
				flash[:notice] = "Signed in successfully."
				sign_in_and_redirect(:user, user)
			else
				session[:omniauth] = omniauth.except('extra')
				redirect_to new_user_registration_url
			end
		end
	end

!SLIDE
# Co jeśli adres email już istnieje w bazie? #
!SLIDE smaller
	
	@@@ ruby
	user = User.new
	if user.save
		flash[:notice] = "Signed in successfully."
		sign_in_and_redirect(:user, user)
	else
		session[:omniauth] = omniauth.except('extra')
		redirect_to new_user_registration_url
	end
!SLIDE
# Refactor #
!SLIDE smaller

	@@@ ruby
	def user
		@user ||= current_user || 
		Student.find_by_identification(uid, provider) || 
		User.find_active_by_email(email) || 
		User.create!(:email => email)
	end
	
	def create
		user.set_identification(uid, provider, token)
		self.current_user = user
		redirect_to somewhere
	end
!SLIDE smaller

	@@@ ruby
	def create
		create_for_current_user || create_for_identification || 
		create_for_existing_mail || create_for_new_user

		@user.set_identification(uid, provder, token) if @user
		self.current_user = @user

		redirect_to @redirect_to
	end
	
!SLIDE smaller
	
	@@@ ruby
	def create_for_current_user
		@user = current_user
		return false unless @user

		@redirect_to = identifications_path
	end

	def create_for_identification
		@user = User.find_by_identification(uid, provider)
		return false unless @user

		flash[:notice] = "Logged in successfully"
		@redirect_to = student_job_postings_path
	end
	
!SLIDE smaller

	@@@ ruby
	def create
		create_for_current_user || create_for_identification || 
		create_for_existing_mail || create_for_new_user

		@user.set_identification(uid, provder, token) if @user
		self.current_user = @user

		redirect_to @redirect_to
	end
!SLIDE smaller

	@@@ ruby
	def create
		create_for_current_user or create_for_identification or 
		create_for_existing_mail or create_for_new_user

		@user.set_identification(uid, provder, token) if @user
		self.current_user = @user

		redirect_to @redirect_to
	end
!SLIDE
# Co jeśli adres email już istnieje w bazie? #
!SLIDE smaller

	@@@ ruby
	
	def create_for_existing_mail
		@user = User.find_active_by_email(email)
		return false unless @user
		
		@redirect_to = root_path
	end
	
!SLIDE smaller

	@@@ ruby

	def create_for_existing_mail
		@user = User.find_active_by_email(email)
		return false unless @user

		flash[:error] = 'Account with this email is already created'
		cookies[:facebook_try] = {:value => true, 
                              :expires => 1.week.from_now}
		@user = nil
		@redirect_to = login_path
	end
!SLIDE
#Pozostałe akcje kontrolera#
!SLIDE smaller

	@@@ ruby
	def index
		@identifications = current_user.identifications
	end

	def destroy
		current_user.identifications.destroy(params[:id])
		flash[:notice] = "Identification was deleted"
		redirect_to identifications_path
	end