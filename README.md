## Final DB Design


## Users Table  

```sql
Table users{
	id SERIAL,
	created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
	username VARCHAR(30) NOT NULL,
	bio VARCHAR(400),
	avatar VARCHAR(200),
	phone VARCHAR(25),
	email VARCHAR(40),
	password VARCHAR(50),
	status VARCHAR(15),
}	
```

We need to ensure either phone or email exists. If the user chooses to sign up with an email, there should be a password associated with the email.  The `CHECK` constrainst would be:

```sql
CHECK(
	(email IS NOT NULL OR phone IS NOT NULL) AND 
    	( email IS NULL OR 
    	(email IS NOT NULL AND password IS NOT NULL))
)
```

## Posts Table

```sql
TABLE posts(
	id SERIAL PRIMARY KEY,
	created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
	url VARCHAR(200) NOT NULL,
	caption VARCHAR(240),
	lat REAL,
	lng REAL
)
```

When user create a post, they can either choose to associate a location with the post or not. If the user chooses to associate a location, we need to ensure that both the latitude and longitude is set. we also need to make sure that the latitude and longitude provided is valid.

The `CHECK` constraint for `lat` would be:

```sql
lat REAL CHECK(lat IS NULL OR ( lat >= -90 AND lat <= 90))
```

The `CHECK` constraint for `lng` would be:

```sql
lng REAL CHECK(lng IS NULL OR (lng >= -180 AND lng <= 180))
```

Associating the post with an user:

```sql
user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE
```

## Comments Table

```sql
TABLE comments (
    id SERIAL PRIMATY KEY,
    created_at TIMESTAMP WITH TIME ZONE DEFUALT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFUALT CURRENT_TIMESTAMP,
    contents VARCHAR(240) NOT NULL
)    
```

Associating a comment with an user(the author of the comment):

```sql
user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE
``` 

Associating a comment with a post(post below which the comment was put):

```sql
post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE
``` 

## Likes Table

```sql
TABLE likes (
    id SERIAL PRIMARY KEY,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
)
```

Associating an user with a like:

```sql
user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE
```

Now, a user can like a comment, a port or both. Associating a post or comment with a like:

```sql
post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
comment_id INTEGER REFERENCES comments(id) ON DELETE CASCADE
```

As we're accounting for like on a comment & like on a post in a single table, we need to add a `CHECK` constraint. The approach would be to ensure that only a single value among the `post_id` and `comment_id` can be non null:

```sql
CHECK(
    (post_id IS NOT NULL AND comment_id IS NULL)
    OR
    (post_id IS NULL AND comment_id IS NOT NULL)
)
```

To complete the idea, we're going to make sure that each combination of `user_id`, `post_id`, `comment_id` is unique:

```sql
UNIQUE(user_id, post_id, comment_id)
```

For Ex: If an user with id 1 liked a comment with id 4 under a post with id 10,
two rows will be created in db with corresponding (user_id, post_id, comment_id) values as (1, 10, NULL) and (1, NULL, 4).