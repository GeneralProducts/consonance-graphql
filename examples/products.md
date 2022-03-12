# Description
#
# Select products and some of their attributes
#
# Explanation
#
# All products are retrieved, with their id, publication date, and ISBN
# The ISBN is a composite type which can return different versions of
# the value, and here we have selected the ISBN13 and ISBN10, possibly for
# indexing purposes on a website, and the ISBN13 formatted with dashes,
# possibly for display purposes.
{
  products {
    id
		publicationDate
    isbn {
			isbn13
			isbn10
			isbnWithDashes
		}
  }
}
