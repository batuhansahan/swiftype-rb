Swiftype Ruby Client (beta)
===

The official [Swiftype](http://swiftype.com) Ruby client for communicating with the Swiftype API. Learn more about Swiftype by visiting [swiftype.com](http://swiftype.com) and creating an account.


Prerequisites
---
1. A Swiftype account. Sign up at [swiftype.com](http://swiftype.com).
2. A compatible Ruby environment.


Installation
---

For now, just clone this repository and then pull in the library:

	rake build && rake install
	
	require 'swiftype'


Overview
---

The client has a few basic methods on `Swiftype` for dealing with `Engines`.  Beyond that, you can perform standard (CRUD) operations on any of the resources.

### Resources

#### Engine

`Engines` are the top-level objects in Swiftype.  They have a free-form `name` field that is translated into a `slug` identifier.

#### Document Type

`DocumentTypes` specify the structure of a set of documents in the `Engine` and are the entry point for searches.  There are three types of fields on a `DocumentType`: `:string`, `:text`, `:enum`, `:integer`, `:float`, and `:date`.

`:string` is for short strings that can be matched in both prefix and full-text searches.  _Example: Chapter titles in a book._

`:text` can be long strings.  They are meant for full-text searches only and will not be used for prefix queries.  _Example: Entire text of an essay._

`:enum` are string traits of a document.  They are not analyzed in any way, and thus can be used to filter and sort queries.  _Example: Hardcover or paperback._

`:date` are ISO 8601 compatible time strings.  They can also be used to filter and sort queries.



#### Document

`Documents` represent all of the pieces of content in an `Engine`.  They are children of a `DocumentType` and conform to its field specification (note: you do *not* need to specify the fields ahead of time, they will be inferred by the contents of a document).  When you perform a search on a `DocumentType`, you will receive `Document` results.  `external_id` is the only required field for a `Document`. It can be any value, such as a numeric ID.



Basic Usage
===

Configuration:
---

Before issuing commands to the API, configure the client with your authentication credentials:

	Swiftype.configure do |config|
	  config.username = "{YOUR_SWIFTYPE_LOGIN}"
	  config.password = "{YOUR_SWIFTYPE_PASSWORD}"
	end

Indexing:
---

#### Engines:

Search engines are the top-level container for the objects you wish to search, and most sites will have a single engine. The engines themselves contain one or more document types, each of which contain the documents themselves.

Create a search engine:

	engine = Swiftype::Engine.new(:name => 'bookstore')
	engine.create!

Get a search engine:
  
	Swiftype::Engine.find('bookstore')

Delete a search engine:

	engine = Swiftype::Engine.find('bookstore')
	engine.destroy!
  
  
#### Document Types

Create a `document_type`:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.create_document_type(:name => 'books')

Get a `document_type`:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
  
Delete a `document_type`. Deleting a `document_type` will also delete every `document` contained within it:

	engine = Swiftype::Engine.find('bookstore')
	engine.destroy_document_type('books')

or, alternatively, call destroy on the `document_type` itself:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	type.destroy!
  

#### Documents

Create a `document`:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	type.create_document({
		:external_id => '1', 
		:fields => [
			{:name => 'title', :value => 'Information Retrieval', :type => 'string'}, 
			{:name => 'genre', :value => 'non-fiction', :type => 'enum'}, 
			{:name => 'author', :value => 'Stefan Buttcher', :type => 'string'}, 
			{:name => 'in_stock', :value => true, :type => 'enum'},
			{:name => 'on_sale', :value => false, :type => 'enum'}
		]});

Get a `document`:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	doc = type.document('1')

Get every `document` within a `document_type`:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	type.documents
  
Update field(s) of a `document`:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	doc = type.document('1')
	doc.update_fields({:in_stock => false })

or, alternatively, update a `document` without retrieving it first:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	type.update_document(:external_id => '1', :fields => { :in_stock => false })

you can also update multiple fields in the same call:

	doc.update_fields({:in_stock => false, :on_sale => true })
  
Delete a `document`:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	doc = type.document('1')
	doc.destroy!

or, alternatively, delete a `document` without retrieving it first:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	type.destroy_document('1')


#### Bulk Operations

Bulk operations will allow you to perform rapid indexing updates and avoid the latency overhead of making repeated requests.

Create `document`s in bulk:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	type.create_documents([{
		:external_id => '2', 
		:fields => [
			{:name => 'title', :value => 'Lucene in Action', :type => 'string'}, 
			{:name => 'genre', :value => 'non-fiction', :type => 'enum'}, 
			{:name => 'author', :value => 'Michael McCandless', :type => 'string'}, 
			{:name => 'in_stock', :value => true, :type => 'enum'},
			{:name => 'on_sale', :value => false, :type => 'enum'}
		]},{
		:external_id => '3', 
		:fields => [
			{:name => 'title', :value => 'MongoDB in Action', :type => 'string'}, 
			{:name => 'genre', :value => 'non-fiction', :type => 'enum'}, 
			{:name => 'author', :value => 'Kyle Banker', :type => 'string'}, 
			{:name => 'in_stock', :value => true, :type => 'enum'},
			{:name => 'on_sale', :value => false, :type => 'enum'}
		]},{
		:external_id => '4', 
		:fields => [
			{:name => 'title', :value => 'The Great Gatsby', :type => 'string'}, 
			{:name => 'genre', :value => 'fiction', :type => 'enum'}, 
			{:name => 'author', :value => 'F. Scott Fitzgerald', :type => 'string'}, 
			{:name => 'in_stock', :value => true, :type => 'enum'},
			{:name => 'on_sale', :value => false, :type => 'enum'}
		]}
	])

Update `document`s in bulk:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	type.update_documents([
		{:external_id => '2', :fields => {:in_stock => 'false'}},
	 	{:external_id => '3', :fields => {:in_stock => 'false'}}
	])

Delete `document`s in bulk:

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	type.destroy_documents(['1','2','3','4'])


Searching:
---

#### Full text search

Search a `document_type` from the query "action":

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	results = type.search("action")


#### Autocomplete

Get autocomplete suggestions from a `document_type` for the prefix "act"

	engine = Swiftype::Engine.find('bookstore')
	type = engine.document_type('books')
	results = type.suggest("act")


Simple Client:
===
The Simple Client is a convenience class that gives you basic, direct access to the Swiftype REST API, without mapping each call to the intermediate objects seen in the examples above. These methods will be more performant, because they avoid unnecessary round-trips to the server, but you will also have to provide more information to each call. Choose whatever suites your use-case.

#### Create a Simple Client

	client = Swiftype::Easy.new

#### Search

	results = client.search('bookstore','books',{SEARCH QUERY})

#### Autocomplete

	results = client.suggest('bookstore','books',{AUTOCOMPLETE PREFIX QUERY})

#### Engines

	client.engines # retrieves every engine
	client.create_engine('bookstore')
	client.destroy_engine('bookstore')

#### Document Types

	client.document_types('bookstore')
	client.create_document_type('bookstore', 'books')
	client.destroy_document_type('bookstore', 'books')

#### Documents
	
	# retrieve all documents
	client.documents('bookstore', 'books')

	# create a document
	client.create_document('bookstore', 'books', {
		:external_id => '1', 
		:fields => [
			{:name => 'title', :value => 'Information Retrieval', :type => 'string'}, 
			{:name => 'genre', :value => 'non-fiction', :type => 'enum'}, 
			{:name => 'author', :value => 'Stefan Buttcher', :type => 'string'}, 
			{:name => 'in_stock', :value => true, :type => 'enum'},
			{:name => 'on_sale', :value => false, :type => 'enum'}
		]})

	# create documents in bulk
	client.create_documents('bookstore', 'books', [{
		:external_id => '2', 
		:fields => [
			{:name => 'title', :value => 'Lucene in Action', :type => 'string'}, 
			{:name => 'genre', :value => 'non-fiction', :type => 'enum'}, 
			{:name => 'author', :value => 'Michael McCandless', :type => 'string'}, 
			{:name => 'in_stock', :value => true, :type => 'enum'},
			{:name => 'on_sale', :value => false, :type => 'enum'}
		]},{
		:external_id => '3', 
		:fields => [
			{:name => 'title', :value => 'MongoDB in Action', :type => 'string'}, 
			{:name => 'genre', :value => 'non-fiction', :type => 'enum'}, 
			{:name => 'author', :value => 'Kyle Banker', :type => 'string'}, 
			{:name => 'in_stock', :value => true, :type => 'enum'},
			{:name => 'on_sale', :value => false, :type => 'enum'}
		]}])

	# update a document
	client.update_document('bookstore','books','1', { :in_stock => false })

	# update documents in bulk
	client.update_documents('bookstore','books', [
		{:external_id => '2', :fields => {:in_stock => false}},
		{:external_id => '3', :fields => {:in_stock => true}}
	])

	# destroy a document
	client.destroy_document('bookstore','books','1')

	# destroy documents in bulk
	client.destroy_documents('bookstore','books',['1','2','3'])




Todo
===

+ Proper response code handling for non-successful requests.
+ Publish gem to rubygems.org
+ Tests!


Questions?
===
Get in touch! We would be happy to help you get up and running.

[Quin](mailto:quin@swiftype.com) and [Matt](mailto:matt@swiftype.com) from [Swiftype](http://swiftype.com)