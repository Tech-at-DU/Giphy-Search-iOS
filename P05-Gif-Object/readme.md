
To finish off our network layer we will need to make our own **Gif Object** that can decode the response from Giphy:

```JSON
"data": [
    {
      "type": "gif",
      "id": "bbshzgyFQDqPHXBo4c",
      "url": "https://giphy.com/gifs/morning-perfect-loops-bbshzgyFQDqPHXBo4c",
      "import_datetime": "2018-03-02 02:30:04",
      "trending_datetime": "2019-09-13 16:30:13",
      "images": {
        "original": {
          "frames": "60",
          "hash": "d1671398a51571d549cd24351d5f725d",
          "height": "480",
          "mp4": "https://media3.giphy.com/media/bbshzgyFQDqPHXBo4c/giphy.mp4?cid=c75b6a02f7ad60b4f7f98b826a7838f4a9270d23555cfb67&rid=giphy.mp4",
          "mp4_size": "725995",
          "size": "4339772",
          "url": "https://media3.giphy.com/media/bbshzgyFQDqPHXBo4c/giphy.gif?cid=c75b6a02f7ad60b4f7f98b826a7838f4a9270d23555cfb67&rid=giphy.gif",
          "webp": "https://media3.giphy.com/media/bbshzgyFQDqPHXBo4c/giphy.webp?cid=c75b6a02f7ad60b4f7f98b826a7838f4a9270d23555cfb67&rid=giphy.webp",
          "webp_size": "1152446",
          "width": "272"
        }
    }
}
```

# Creating the Gif Object

> [action]
>Create a new swift file called `Gif`.
>
> ![](./assets/GifCreate.png)
>
> This is where we will decode the JSON response from Giphy into an object.
>
> Copy the following code into your **Gif** class.
>
```Swift
import Foundation
/// Array of Gif objects.
struct GifArray: Decodable {
    var gifs: [Gif]
    enum CodingKeys: String, CodingKey {
        case gifs = "data"
    }
}
/// Contains giph properties
struct Gif: Decodable {
    var gifSources: GifImages
    enum CodingKeys: String, CodingKey {
        case gifSources = "images"
    }
    /// Returns download url of the originial gif
    func getGifURL() -> String{
        return gifSources.original.url
    }
}
/// Stores the original Gif
struct GifImages: Decodable {
    var original: original
    enum CodingKeys: String, CodingKey {
        case original = "original"
    }
}
/// URL to data of Gif
struct original: Decodable {
    var url: String
}
```
>
> The object has multiple objects inside of it. Lets go over them.
>
> `GifArray` this decodes the data for the key **data** into an array of Gif objects.
>
> `Gif` this is the actual gif object that contains the images as well as a function to return the url for it.
>
> `GifImages` decodes the data for different sources of the Gif.
>
> `original` decodes the data from the **original** key and stores the URL.
>

# Understanding Decodable

> [action]
> Decodable allows us to create custom objects that can easily fetch data from a JSON response.
>
> **How it works** Decodable looks for keys that you define and fetches the value for it from the JSON object. The value is then stored inside the custom object that you created.
>
> To go more in depth with decodable take a look [Apples Documentation](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types)
>

# Decoding NSData into our new Gif object

Now that we've created our Gif Object lets link it to our URLSession task.

> [action]
>Add the following code to the `fetchGifs` function inside our **GifNetwork**.
>
```Swift
/**
Fetches gifs from the Giphy api
-Parameter searchTerm: What  we should query gifs of.
-Returns: Optional array of gifs
*/
func fetchGifs(searchTerm: String, completion: @escaping (_ response: GifArray?) -> Void) {
    // Create a GET url request
    let url = URL(string: "https://api.giphy.com/v1/gifs/search?api_key=\(apiKey)&q=\(searchTerm)")!
    var request = URLRequest(url: url)
    request.httpMethod = "GET"
    URLSession.shared.dataTask(with: request) { (data, response, error) in
        if let err = error {
            print("Error fetching from Giphy: ", err.localizedDescription)
        }
        do {
            // Decode the data into array of Gifs
            DispatchQueue.main.async {
                let object = try! JSONDecoder().decode(GifArray.self, from: data!)
                completion(object)
            }
        }
    }.resume()
}
```

> [Action] The ViewController needs to store an array of Gifs. Add a variable for this at the top of ViewController.swift, below `var network = GifNetwork()`:

```Swift
var gifs: [Gif] = []
```

>
> We now decode the JSON object into our custom Gif Object.
>
> Let's test it out! Navigate back to the **ViewController** class and change the `searchGifs` function to the following:
>
```Swift
    /**
    Fetches gifs based on the search term and populates tableview
    - Parameter searchTerm: The string to search gifs of
    */
    func searchGifs(for searchText: String) {
        network.fetchGifs(searchTerm: searchText) { gifArray in
            if gifArray != nil {
                print(gifArray!.gifs.count)
                self.gifs = gifArray!.gifs
                self.tableView.reloadData()
            }
        }
    }
```
>
> We are using something called a `completion block`. These allow us to return an object once a function has finished executing. They are typically used for network calls or tasks that take a while to complete.
>
Press `Command + R` and run the program. Type something into the search bar and search. Check and see what gets printed out into the console.

Test your project, when it opens in the simulator enter a search term and hit return. You should see something like: 

```Swift
50
```

This is the number of gifs returned for your search. 

This comes from the line `print(gifArray!.gifs.count)` which is printing the count of the `gifArray.gifs`. Change the line to look like this: 

```Swift
print(gifArray!.gifs)
```

This should print something like: 

```
[GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/3pZipqyo1sqHDfJGtz/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/28GHfhGFWpFgsQB4wR/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/5FDfOtafB4Gnwr9dBm/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media1.giphy.com/media/E1w0yvMxBIv5M8WkL8/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/icUEIrjnUuFCWDxFpU/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media0.giphy.com/media/zSHERzpaQ9x8k/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/65OOoKUgwJlJpRma8O/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/3otPongOYeG9iINM8o/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media3.giphy.com/media/QAsHga1AB6dIGUsui6/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/d8oI97avlJAygnp7RC/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/1kJxyyCq9ZHXX0GM3a/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/WOwiryOPA0G6jhKqB0/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/VdAvVcQLJDwKKDUDJR/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/brsEO1JayBVja/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media3.giphy.com/media/xT9IgG50Fb7Mi0prBC/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media1.giphy.com/media/WS3i2y88foYpE584rI/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media3.giphy.com/media/xT0BKIMw1xrdhcqUaA/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/IgGcxqawkRc6y43Z6I/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media0.giphy.com/media/Sg4DwEJrCpGIU/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/Cmr1OMJ2FN0B2/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/dzaUX7CAG0Ihi/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/HOh1tBgpWqtvC9GMD2/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/FBeSx3itXlUQw/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media3.giphy.com/media/dsPBfiEEozyXUXShhB/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media3.giphy.com/media/BVStb13YiR5Qs/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media3.giphy.com/media/ASd0Ukj0y3qMM/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/vFKqnCdLPNOKc/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media0.giphy.com/media/IThjAlJnD9WNO/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media1.giphy.com/media/bcKmIWkUMCjVm/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media1.giphy.com/media/QYkX9IMHthYn0Y3pcG/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/26gYCFcNenajfsnVC/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media0.giphy.com/media/QCDgYrgrfggee4MDPD/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/3o6Ztl7oraKm4ZJ9mw/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media1.giphy.com/media/888R35MJTmDxQfRzfS/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/PnDRNekrgtHh5jXMna/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media3.giphy.com/media/AmWAaenfh2RGw/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/27c7Jo2GU5tpCEQT0y/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media0.giphy.com/media/remMvcYDTQNKE/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/4Nq9NNTuIlMnm/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media0.giphy.com/media/oC5V6VFUiwPjjMN4Xe/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/t4kiu126itmEcriAIH/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media0.giphy.com/media/3o7aCUQ7TsQ3cndExq/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/qQh0DBncuFJwQ/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media1.giphy.com/media/ifxLK48cnyDDi/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media0.giphy.com/media/NZmrvsfxpbTSU/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media1.giphy.com/media/EK24OWrJSy1GkkNu0y/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/Y8ocCgwtdj29O/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media2.giphy.com/media/UtzyBJ9trryNO4R3Ee/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media0.giphy.com/media/9HBduC3ZIgrG8/giphy.gif"))), GiphySearchIOS.Gif(gifSources: GiphySearchIOS.GifImages(original: GiphySearchIOS.original(url: "https://media4.giphy.com/media/n7KBLYG0Yqs0aZa4iP/giphy.gif")))]
```

This is displaying the list of all of the Gifs returned for the results. 

> Awesome! Now we have **URL's that we can use to populate our tableView**.

# Refactor

Before we continue let's add a couple things to perfect our network layer

> [action]
>
> To finish off our network layer we will add in a `urlBuilder`. urlBuilders create a custom URL based on query items
>
> Add the following code under your `fetchGifs` function in the `GifNetwork` class.
>
```Swift
/**
    Returns a url with our API key and search term
    - Parameter searchTerm: The string to search gifs of
    - Returns: URL of search term & api key
    */
    func urlBuilder(searchTerm: String) -> URL {
        let apikey = apiKey
        var components = URLComponents()
           components.scheme = "https"
           components.host = "api.giphy.com"
           components.path = "/v1/gifs/search"
           components.queryItems = [
               URLQueryItem(name: "api_key", value: apikey),
               URLQueryItem(name: "q", value: searchTerm),
               URLQueryItem(name: "limit", value: "1") // Edit limit to display more gifs
           ]
        return components.url!
    }
```
>
> In the fetchGifs function change: `let url = URL(string: "https://api.giphy.com/v1/gifs/search?api_key=\(apiKey)&q=\(searchTerm)")!` to use our URL builder: `let url = urlBuilder(searchTerm: searchTerm)`
>
> **Next** - We need to save our gifs inside our **ViewController**. Navigate back to our ViewController and underneath `var network = GifNetwork()` put the following code: `var gifs = [Gif]()`.
>
> Lastly lets edit out `searchGifs` function to save the gifs to our new gifs object.
>
> Replace the function with the following code:
>
```Swift
/**
    Fetches gifs based on the search term and populates tableview
    - Parameter searchTerm: The string to search gifs of
    */
    func searchGifs(for searchText: String) {
        network.fetchGifs(searchTerm: searchText) { gifArray in
            if gifArray != nil {
                self.gifs = gifArray!.gifs
                self.tableView.reloadData()
            }
        }
    }
```
>
> Awesome, now we are ready to populate the tableView with gifs!
>

# Next Steps

Now that we have our network finished we can go ahead and populate our tableView based on our search query.

- [P06-Displaying-Gifs](./P06-Displaying-Gifs)
