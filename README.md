Fork of Tcl rest client module.
Original documentation at https://core.tcl-lang.org/tcllib/doc/trunk/embedded/md/tcllib/files/modules/rest/rest.md

## Changes

### copy
  The copy option can recursively copy other definitions.
  Support for multiple inheritance has been added.

### child_url
  The child_url value is joined with the copied url value.

### Appending options to *_args
  static_args, opt_args, and req_args support appending values.
  If any of these keys start with a + (e.g., +static_args, +opt_args, +req_args), the values are appended to the copied options.

### Templating
  Array names starting with @ are used as templates and can be copied. Interfaces is not created for such definitions.

### Authentication
 A new authentication type, **command**, has been added.
 Authentication is initialized after the interface is created, using: **{interface_namespace}::::command_auth** *{tcl_command}*



## Example

```tcl
# youtube base url template
set youtube(@BaseUrl) {
	url https://youtube.googleapis.com/youtube/v3
	format json
	auth command
	error-body 1
}

# youtube http get template
set youtube(@BaseGet) {
	copy @BaseUrl
	method get
	static_args {
		part snippet
	}
	opt_args {
		pageToken:
		maxResults:50
	}
}

# youtube http post template
set youtube(@BasePost) {
	copy @BaseUrl
	method post
	headers {
		content-type application/json
	}
}

# playlistItems url template
set youtube(@BasePlaylistItemUrl) {
	child_url playlistItems
}

# playlistItems get interface.
# multiple recursive inheritance is used 
# part is appended to opt_args
set youtube(playlistItems) {
	copy {@BaseGet @BasePlaylistItemUrl}
	unset static_args
	+opt_args {
		part:contentDetails
	}
	req_args {
		playlistId:
	}
}

# create playlist item
set youtube(createPlaylistItem) {
	copy {@BasePost @BasePlaylistItemUrl}
	static_args {
		part snippet
	}
}


rest::create_interface youtube
youtube::command_auth [list [info class instances google::oauth2] get_access_token_and_type]
```
