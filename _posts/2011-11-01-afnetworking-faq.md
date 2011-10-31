---
layout: post
title: "AFNetworking FAQ"
author: mt
date: 2011-11-01
published: false
---

In the course of developing and maintaining [AFNetworking](https://github.com/gowalla/AFNetworking), I've come across all sorts of questions about it. It's been my sincere pleasure to answer as many of those as I can, and I want to thank everyone who has reached out; it's been unbelievably helpful to me in developing the library.

I have noticed that a few questions come up with some frequency.  One may go so far as to call them "Frequently Asked Questions". Anyway, I thought it might be useful to answer these all in one place, for the benefit of anyone else who might have these same questions:

## How do I upload a file?

Uploading a file is really just constructing an HTTP multi-part form body for a URL request.`AFHTTPClient` provides several helper functions that make this easy.

To create the request object, you would do something like:

	NSData *imageData = UIImagePNGRepresentation(image);
	NSURLRequest *request = [client multipartFormRequestWithMethod:@"POST" path:@"/upload" parameters:nil constructingBodyWithBlock: ^(id <AFMultipartFormData> formData) {
		[formData appendPartWithFileData:imageData mimeType:@"image/png" name:@"avatar"];
	}];

You could then pass `request` into any request operation, like `AFJSONRequestOperation -JSONRequestOperationWithRequest:success:failure:`.

## Does AFNetworking have any cacheing mechanisms built-in?

AFNetworking takes advantage of the cacheing functionality already provided by `NSURLCache` and any of its subclasses.

We strongly recommend you try out [Pete Steinberger's fork of SDURLCache](https://github.com/steipete/SDURLCache), which provides disk cacheing, which is not otherwise implemented by `NSURLCache` on iOS.

## How do you keep requests running in the background after the app has been dismissed?

Any request operations running or enqueued in an `NSOperationQueue` will continue to run if you kick off a background task in your AppDelegate's `-applicationWillResignActive:` method. 

You should set the expiration handler to cancel any remaining operations for an HTTP client or other operation queues for example:

	- (void)applicationWillResignActive:(UIApplication *)application {
    	[application beginBackgroundTaskWithExpirationHandler:^(void) {
			[networkOperationQueue cancelAllOperations];
        	[[YourHTTPClient sharedClient] cancelAllHTTPOperations];
    	}];
	}

## How do I send JSON parameters in my request?

If you're using `AFHTTPClient`, set the `parameterEncoding` property to `AFJSONParameterEncoding`. Any method on that HTTP client with a `parameters` argument will now encode the passed object into a JSON string and set the HTTP body and `Content-Type` header appropriately.

Otherwise, you can do this manually by adding the header `Content-Type: application/json`, and setting the body of your request to a JSON string.

## How do I track upload or download progress?

`AFURLConnectionOperation` provides the methods `setUploadProgressBlock:` and `setDownloadProgressBlock:`. Each method takes a single parameter, which is a block that will be executed periodically during the lifetime of the request. The block has no return type and takes 3 arguments: the number of bytes read or written for this callback, the total number of bytes read or written so far, and the total number of bytes expected to be read or written.

If you wanted to update a progress bar, you could set the respective progress block to set the progress amount to total number bytes read or written divided by the expected number, normalized between 0.0 and 1.0. UI updates based on this value will update asynchronously as the request is being made.

## How do I make streaming requests?

All request operations have the `inputStream` and `outputStream` property. To download data as its sent from the server, set the `outputStream` property with an `NSOutputStream` object,  To upload the contents of a file, for instance, by streaming it into the body of a request, set `inputStream` with an `NSInputStream` object.

## What's with the "AF" prefix?

The "AF" in AFNetworking stands for "Alamofire", which is the former name of the company known today as Gowalla. Alamofire Inc. was named after the [Alamo Fire](http://aggie-horticulture.tamu.edu/wildseed/alamofire.html), a hybrid of [the Bluebonnet](http://en.wikipedia.org/wiki/Bluebonnet_(plant))--Texas' state flower.

Using AF is also a nod to the "NS" prefix used in Apple's Foundation framework, which harkens back to its [NeXTSTEP](http://en.wikipedia.org/wiki/NeXTSTEP) roots.
