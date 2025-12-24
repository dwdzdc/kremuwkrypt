# Kremuwkrypt

## Overview of functionalities
A gallery application with an e2e encryption based on AES.
Anyone can create a gallery and upload the pictures and clips in there.
On gallery creation they get an url to their gallery.
An URL is a combination of access and a key.
A key is AES key after '#' that allows for encrypting and decrypting the data in gallery. It is accessible to read only for frontend, it is never sent to the server effectively making e2ee possible with a browser without an account.
The access is a set of permissions associated with the URL. It can be view, write and admin. View allows for read only of the content in the gallery, write can also write and admin can also manage the other accesses: create, delete, view etc.
It is possible to delete all accesses to a gallery, and that effectively makes gallery unaccessible forever so the gallery and a content associated with it is going to be deleted along with the last access.
The file storage on the server side can be configured as local fs storage or s3.
Picture upload: a WASM component encrypts the original picture and produces thumbnails, then all the ecnrypted content is sent to backend for storage.
The metadata of the picture is never removed and available on thumbnails and original image, but encrypted with them.
Video upload: a WASM component encrypts the video and captures the first frame and produces thumbnails out of it, then all the encrypted content is sent to backend just like with pictures.
Gallery browse: a klient will make a request to API to fetch a list of existing objects and then for each object they will fetch a thumbnail of appropriate size. On click there will be bigger, but not original version displayed. But each item will also have a button to see the original image.

## Assumptions
The project is opensource
The project allows for privacy safe media sharing
The project is intended for small scale usage and doesn't scale horizontally, but rather is self contained in one docker image (we can change that in the future)
The project is educational really. It could be made with different technology perhaps but our personal goal is to learn Rust

## Backend API proposition
POST /gallery: create gallery
req: {encrypted_text}
resp: 200 {gallery_id, access_id}
comm: the application frontend will compose a link out of gallery_id and an access_id. The access is admin. The AES key is frontend-generated. The encrypted text is for testing if the visitor has a correct key (to prevent using different keys in one gallery).

GET /gallery/{gallery_id}/{access_id}
resp: 200 {[encrypted_media_uri]}
comm: returns a list of media in uri. Each uri is for original file but it should be possible to just add query param like ?thumbnail=100 to it.

POST /gallery/{gallery_id}/{access_id}
req: {original: b64_original, thumbnail_X: b64_thumbnail_X}
resp: 200 {uri}
comm: upload a single file and its thumbnails.


GET /media/{media_id}/{?thumbnail_size}
resp: 200 encrypted file
comm: fetch file or its thumbnail of given size. Note: maybe instead of file we should return a redirect to an s3 url? So that the download omits the app. A performance improvement.


POST /gallery/{gallery_id}/{access_id}/access
req: {access_type["admin", "viewer", "writer"]}
resp: {access_id}
comm: create a new access


DELETE /gallery/{gallery_id}/{access_id}/{target_access_id}
resp: 204
comm: remove access

 
