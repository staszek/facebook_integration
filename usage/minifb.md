!SLIDE
# Przykład użycia #
## Wiadomość na ścianie użytkownika ##
!SLIDE
# MiniFb gem #
## https://github.com/appoxy/mini_fb ##
!SLIDE smaller

	@@@ ruby
	
		MiniFB.post(current_user.facebook_token, 'me',
			{ :type => 'feed',
				:link => root_path,
				:message => 'has posted a wall message',
				:name => current_user.full_name,
				:caption => 'site.de',
				:description => 'description',
				:picture => 'www.file.com/example.png'
			})
