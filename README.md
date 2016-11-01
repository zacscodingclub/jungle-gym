# Jungle Gym
A web app for Amazon product research.  

* http://docs.aws.amazon.com/AWSECommerceService/latest/DG/RG_OfferSummary.html
* https://hashrocket.com/blog/posts/how-to-make-rails-5-api-only
* http://sourcey.com/building-the-prefect-rails-5-api-only-app/


## Tasks
1. Figure out Request/Response using Vacuum gem
  * Format Amazon request
  ```ruby
  request = Vacuum.new
  # In production, I need to export sensitive keys and associate tag
  request.associate_tag = ENV['AWS_ASSOCIATE_TAG']
  response = request.item_search(
    query: {
      'Keywords' => 'garlic press',
      'ResponseGroup' => 'Medium,OfferSummary,SalesRank',
      'SearchIndex' => 'All'
    }
  )

  ```
  * Handle response
    * What info do I need?
      1. ~~Initial: ASIN, Title, Brand, Sellers, Price, Category, Rank~~
      2. Future: Features, Variations, Parse reviews and add that info

  ```ruby
  hash_response = response.to_h

  # needs error handling
  items_array = hash_response["ItemSearchResponse"]["Items"]["Item"]

  items_array.each do |item|
    Product.create(
         :asin => item["ASIN"],
        :title => item["ItemAttributes"]["Title"],
        :brand => item["ItemAttributes"]["Brand"],
      :sellers => item["OfferSummary"]["TotalNew"],
        :price => item["OfferSummary"]["LowestNewPrice"]["Amount"],
     :category => item["ItemAttributes"]["ProductGroup"],
   :sales_rank => item["SalesRank"]
   )
  end

  items_array[0] =>
  {
      "ASIN" => "B00I937QEI",
      "DetailPageURL" => "https://www.amazon.com/Alpha-Grillers-Garlic-Stainless-Silicone/dp/B00I937QEI%3FSubscriptionId%3DAKIAI2LDUWLVULJIKCPA%26tag%3Dzbaston-20%26linkCode%3Dxm2%26camp%3D2025%26creative%3D165953%26creativeASIN%3DB00I937QEI",
      "ItemLinks" => {
          "ItemLink" => [{
              "Description" => "Technical Details", "URL" => "https://www.amazon.com/Alpha-Grillers-Garlic-Stainless-Silicone/dp/tech-data/B00I937QEI%3FSubscriptionId%3DAKIAI2LDUWLVULJIKCPA%26tag%3Dzbaston-20%26linkCode%3Dxm2%26camp%3D2025%26creative%3D386001%26creativeASIN%3DB00I937QEI"
          }, {
              "Description" => "Add To Baby Registry", "URL" => "https://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3DB00I937QEI%26SubscriptionId%3DAKIAI2LDUWLVULJIKCPA%26tag%3Dzbaston-20%26linkCode%3Dxm2%26camp%3D2025%26creative%3D386001%26creativeASIN%3DB00I937QEI"
          }, {
              "Description" => "Add To Wedding Registry", "URL" => "https://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3DB00I937QEI%26SubscriptionId%3DAKIAI2LDUWLVULJIKCPA%26tag%3Dzbaston-20%26linkCode%3Dxm2%26camp%3D2025%26creative%3D386001%26creativeASIN%3DB00I937QEI"
          }, {
              "Description" => "Add To Wishlist", "URL" => "https://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3DB00I937QEI%26SubscriptionId%3DAKIAI2LDUWLVULJIKCPA%26tag%3Dzbaston-20%26linkCode%3Dxm2%26camp%3D2025%26creative%3D386001%26creativeASIN%3DB00I937QEI"
          }, {
              "Description" => "Tell A Friend", "URL" => "https://www.amazon.com/gp/pdp/taf/B00I937QEI%3FSubscriptionId%3DAKIAI2LDUWLVULJIKCPA%26tag%3Dzbaston-20%26linkCode%3Dxm2%26camp%3D2025%26creative%3D386001%26creativeASIN%3DB00I937QEI"
          }, {
              "Description" => "All Customer Reviews", "URL" => "https://www.amazon.com/review/product/B00I937QEI%3FSubscriptionId%3DAKIAI2LDUWLVULJIKCPA%26tag%3Dzbaston-20%26linkCode%3Dxm2%26camp%3D2025%26creative%3D386001%26creativeASIN%3DB00I937QEI"
          }, {
              "Description" => "All Offers", "URL" => "https://www.amazon.com/gp/offer-listing/B00I937QEI%3FSubscriptionId%3DAKIAI2LDUWLVULJIKCPA%26tag%3Dzbaston-20%26linkCode%3Dxm2%26camp%3D2025%26creative%3D386001%26creativeASIN%3DB00I937QEI"
          }]
      },
      "SalesRank" => "299",
      "SmallImage" => {
          "URL" => "http://ecx.images-amazon.com/images/I/51IE7kxlsYL._SL75_.jpg", "Height" => {
              "__content__" => "75", "Units" => "pixels"
          }, "Width" => {
              "__content__" => "75", "Units" => "pixels"
          }
      }, "MediumImage" => {
          "URL" => "http://ecx.images-amazon.com/images/I/51IE7kxlsYL._SL160_.jpg", "Height" => {
              "__content__" => "160", "Units" => "pixels"
          }, "Width" => {
              "__content__" => "160", "Units" => "pixels"
          }
      }, "LargeImage" => {
          "URL" => "http://ecx.images-amazon.com/images/I/51IE7kxlsYL.jpg", "Height" => {
              "__content__" => "500", "Units" => "pixels"
          }, "Width" => {
              "__content__" => "500", "Units" => "pixels"
          }
      }, "ImageSets" => {
          "ImageSet" => [{
              "SwatchImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51CoXPaalgL._SL30_.jpg", "Height" => {
                      "__content__" => "30", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "30", "Units" => "pixels"
                  }
              }, "SmallImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51CoXPaalgL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "ThumbnailImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51CoXPaalgL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "TinyImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51CoXPaalgL._SL110_.jpg", "Height" => {
                      "__content__" => "110", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "110", "Units" => "pixels"
                  }
              }, "MediumImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51CoXPaalgL._SL160_.jpg", "Height" => {
                      "__content__" => "160", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "160", "Units" => "pixels"
                  }
              }, "LargeImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51CoXPaalgL.jpg", "Height" => {
                      "__content__" => "500", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "500", "Units" => "pixels"
                  }
              }, "HiResImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/81ANFfGJr%2BL.jpg", "Height" => {
                      "__content__" => "1500", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "1500", "Units" => "pixels"
                  }
              }, "Category" => "variant"
          }, {
              "SwatchImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/417yywTETUL._SL30_.jpg", "Height" => {
                      "__content__" => "30", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "30", "Units" => "pixels"
                  }
              }, "SmallImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/417yywTETUL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "ThumbnailImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/417yywTETUL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "TinyImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/417yywTETUL._SL110_.jpg", "Height" => {
                      "__content__" => "110", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "110", "Units" => "pixels"
                  }
              }, "MediumImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/417yywTETUL._SL160_.jpg", "Height" => {
                      "__content__" => "160", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "160", "Units" => "pixels"
                  }
              }, "LargeImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/417yywTETUL.jpg", "Height" => {
                      "__content__" => "500", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "500", "Units" => "pixels"
                  }
              }, "HiResImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/81YTprr%2BmwL.jpg", "Height" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }
              }, "Category" => "variant"
          }, {
              "SwatchImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51Yk-V5PhiL._SL30_.jpg", "Height" => {
                      "__content__" => "30", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "30", "Units" => "pixels"
                  }
              }, "SmallImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51Yk-V5PhiL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "ThumbnailImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51Yk-V5PhiL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "TinyImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51Yk-V5PhiL._SL110_.jpg", "Height" => {
                      "__content__" => "110", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "110", "Units" => "pixels"
                  }
              }, "MediumImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51Yk-V5PhiL._SL160_.jpg", "Height" => {
                      "__content__" => "160", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "160", "Units" => "pixels"
                  }
              }, "LargeImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51Yk-V5PhiL.jpg", "Height" => {
                      "__content__" => "500", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "500", "Units" => "pixels"
                  }
              }, "HiResImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/81Acr1MULYL.jpg", "Height" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }
              }, "Category" => "variant"
          }, {
              "SwatchImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41qt%2BbJ8CSL._SL30_.jpg", "Height" => {
                      "__content__" => "30", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "30", "Units" => "pixels"
                  }
              }, "SmallImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41qt%2BbJ8CSL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "ThumbnailImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41qt%2BbJ8CSL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "TinyImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41qt%2BbJ8CSL._SL110_.jpg", "Height" => {
                      "__content__" => "110", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "110", "Units" => "pixels"
                  }
              }, "MediumImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41qt%2BbJ8CSL._SL160_.jpg", "Height" => {
                      "__content__" => "160", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "160", "Units" => "pixels"
                  }
              }, "LargeImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41qt%2BbJ8CSL.jpg", "Height" => {
                      "__content__" => "500", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "500", "Units" => "pixels"
                  }
              }, "HiResImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/812VqeERJjL.jpg", "Height" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }
              }, "Category" => "variant"
          }, {
              "SwatchImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41KkAxpOHNL._SL30_.jpg", "Height" => {
                      "__content__" => "30", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "30", "Units" => "pixels"
                  }
              }, "SmallImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41KkAxpOHNL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "ThumbnailImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41KkAxpOHNL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "TinyImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41KkAxpOHNL._SL110_.jpg", "Height" => {
                      "__content__" => "110", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "110", "Units" => "pixels"
                  }
              }, "MediumImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41KkAxpOHNL._SL160_.jpg", "Height" => {
                      "__content__" => "160", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "160", "Units" => "pixels"
                  }
              }, "LargeImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41KkAxpOHNL.jpg", "Height" => {
                      "__content__" => "500", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "500", "Units" => "pixels"
                  }
              }, "HiResImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/81J8xurvIVL.jpg", "Height" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }
              }, "Category" => "variant"
          }, {
              "SwatchImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41MgKpgEsIL._SL30_.jpg", "Height" => {
                      "__content__" => "30", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "30", "Units" => "pixels"
                  }
              }, "SmallImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41MgKpgEsIL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "ThumbnailImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41MgKpgEsIL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "TinyImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41MgKpgEsIL._SL110_.jpg", "Height" => {
                      "__content__" => "110", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "110", "Units" => "pixels"
                  }
              }, "MediumImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41MgKpgEsIL._SL160_.jpg", "Height" => {
                      "__content__" => "160", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "160", "Units" => "pixels"
                  }
              }, "LargeImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41MgKpgEsIL.jpg", "Height" => {
                      "__content__" => "500", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "500", "Units" => "pixels"
                  }
              }, "HiResImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/81Tn7qUHerL.jpg", "Height" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }
              }, "Category" => "variant"
          }, {
              "SwatchImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51z0PSbUxwL._SL30_.jpg", "Height" => {
                      "__content__" => "28", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "30", "Units" => "pixels"
                  }
              }, "SmallImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51z0PSbUxwL._SL75_.jpg", "Height" => {
                      "__content__" => "70", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "ThumbnailImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51z0PSbUxwL._SL75_.jpg", "Height" => {
                      "__content__" => "70", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "TinyImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51z0PSbUxwL._SL110_.jpg", "Height" => {
                      "__content__" => "103", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "110", "Units" => "pixels"
                  }
              }, "MediumImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51z0PSbUxwL._SL160_.jpg", "Height" => {
                      "__content__" => "149", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "160", "Units" => "pixels"
                  }
              }, "LargeImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51z0PSbUxwL.jpg", "Height" => {
                      "__content__" => "466", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "500", "Units" => "pixels"
                  }
              }, "HiResImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/81xWQD0aLsL.jpg", "Height" => {
                      "__content__" => "2387", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }
              }, "Category" => "variant"
          }, {
              "SwatchImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41T1EiGRi%2BL._SL30_.jpg", "Height" => {
                      "__content__" => "30", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "30", "Units" => "pixels"
                  }
              }, "SmallImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41T1EiGRi%2BL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "ThumbnailImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41T1EiGRi%2BL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "TinyImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41T1EiGRi%2BL._SL110_.jpg", "Height" => {
                      "__content__" => "110", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "110", "Units" => "pixels"
                  }
              }, "MediumImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41T1EiGRi%2BL._SL160_.jpg", "Height" => {
                      "__content__" => "160", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "160", "Units" => "pixels"
                  }
              }, "LargeImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/41T1EiGRi%2BL.jpg", "Height" => {
                      "__content__" => "500", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "500", "Units" => "pixels"
                  }
              }, "HiResImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/81hqQjT2MxL.jpg", "Height" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }
              }, "Category" => "variant"
          }, {
              "SwatchImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51IE7kxlsYL._SL30_.jpg", "Height" => {
                      "__content__" => "30", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "30", "Units" => "pixels"
                  }
              }, "SmallImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51IE7kxlsYL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "ThumbnailImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51IE7kxlsYL._SL75_.jpg", "Height" => {
                      "__content__" => "75", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "75", "Units" => "pixels"
                  }
              }, "TinyImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51IE7kxlsYL._SL110_.jpg", "Height" => {
                      "__content__" => "110", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "110", "Units" => "pixels"
                  }
              }, "MediumImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51IE7kxlsYL._SL160_.jpg", "Height" => {
                      "__content__" => "160", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "160", "Units" => "pixels"
                  }
              }, "LargeImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/51IE7kxlsYL.jpg", "Height" => {
                      "__content__" => "500", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "500", "Units" => "pixels"
                  }
              }, "HiResImage" => {
                  "URL" => "http://ecx.images-amazon.com/images/I/91NyNGROs9L.jpg", "Height" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }, "Width" => {
                      "__content__" => "2560", "Units" => "pixels"
                  }
              }, "Category" => "primary"
          }]
      },
      "ItemAttributes" => {
          "Brand" => "Alpha Grillers",
          "Color" => "Black",
          "EAN" => "0802567697880",
          "EANList" => {
              "EANListElement" => ["0802567697880", "0802567723329", "0802567707084", "0820103971416"]
          },
          "Feature" => ["SOLID STAINLESS STEEL. Since the garlic press is made from solid, high quality stainless steel, and the peeler tube from tough FDA approved silicone, these cooking gadgets will last as long as you need them.", "PRESS UNPEELED GARLIC AND GINGER. Thanks to the heavy duty construction and ergonomic design, you can easily mince both unpeeled garlic cloves and peeled root ginger. What's more, there is next to no waste: garlic comes out, peel stays in.", "EASY SQUEEZE EASY CLEAN. The large, comfortable handles and ingenious design make squeezing a breeze, even for those with weaker grip or small hands. Whats more, clean up is super simple too. Just rinse under water or run through the dishwasher.", "WANT PEELED CLOVES FOR SLICING? NO PROBLEM! Crushed garlic is delicious and versatile, but sometimes you want to slice it up! Being the only press on Amazon to come with a garlic peeler, we've got you covered.", "LIFETIME GUARANTEE: LOVE IT OR YOUR MONEY BACK! We are so confident that you will love our Press & Peeler Set that we are offering all customers a lifetime guarantee. If at any point you decide you are not completely satisfied with your set, just drop us an email and we will refund 100% of the money. You wont get this kind of risk free service with big brands like Rosle, Pampered Chef and Kuhn Rikon."],
          "ItemDimensions" => {
              "Height" => {
                  "__content__" => "138", "Units" => "hundredths-inches"
              }, "Length" => {
                  "__content__" => "748", "Units" => "hundredths-inches"
              }, "Weight" => {
                  "__content__" => "10", "Units" => "ounces"
              }, "Width" => {
                  "__content__" => "256", "Units" => "hundredths-inches"
              }
          },
          "MPN" => "820103971416",
          "PackageDimensions" => {
              "Height" => {
                  "__content__" => "190", "Units" => "hundredths-inches"
              }, "Length" => {
                  "__content__" => "870", "Units" => "hundredths-inches"
              }, "Weight" => {
                  "__content__" => "60", "Units" => "hundredths-pounds"
              }, "Width" => {
                  "__content__" => "400", "Units" => "hundredths-inches"
              }
          }, "PartNumber" => "820103971416", "ProductGroup" => "Kitchen", "ProductTypeName" => "KITCHEN", "Title" => "Alpha Grillers Garlic Press and Peeler Set. Stainless Steel Mincer and Silicone Tube Roller", "UPC" => "802567707084", "UPCList" => {
              "UPCListElement" => ["802567707084", "820103971416", "802567723329", "802567697880"]
          }
      },
      "OfferSummary" => {
          "LowestNewPrice" => {
              "Amount" => "1797", "CurrencyCode" => "USD", "FormattedPrice" => "$17.97"
          }, "LowestUsedPrice" => {
              "Amount" => "1438", "CurrencyCode" => "USD", "FormattedPrice" => "$14.38"
          }, "TotalNew" => "3", "TotalUsed" => "2", "TotalCollectible" => "0", "TotalRefurbished" => "0"
      }, "EditorialReviews" => {
          "EditorialReview" => {
              "Source" => "Product Description", "Content" => "<b>Garlic Lovers - Say Hello To Amazon's No.1 Top Rated Garlic Press!</b> <p> These utensils are built using the highest quality materials: solid stainless steel for the press and durable FDA approved silicone for the peeler. If you are looking for <b>tools that will last a lifetime</b>, you've found them.</p> <p> Add delicious, fresh minced or sliced garlic to all you meals in seconds. Once you <b>discover how quickly and easily you can produce you own fresh garlic mince</b> you'll be adding to everything. Home made garlic bread, rich pasta sauce, mouthwatering hummus, oven roast veggies, mashed potatoes. The options are only limited by your imagination!</p> <p> Not only is each kitchen tool gorgeously designed, they also come in an <b>elegant, fabric lined presentation case</b> (check out the second image on the left). A perfect birthday or holiday present for any men or women who love to cook.</p> <p> You risk absolutely nothing. The Alpha Grillers Garlic Press & Peeler Set is backed by an <b>unconditional 100% lifetime money back guarantee.</b></p> <p> America agrees: with a rating of 4.9 out of 5 stars, you have found the best press on Amazon. Don't settle for anything less. Click the <b>'Add To Cart' button now</b> to order your set today.</p>", "IsLinkSuppressed" => "0"
          }
      }
  }

  ```
2. Construct Quick UI
  * Make a mock up
  * Install Bootstrap

3. Build out controllers and models
  * User
    1. Devise Basic Login info
  * ProductSearch
    1. Model: id, user, keyword used
    2. Controller:
      * Index: Show Users' saved searches
      * Show: Show specific search results
      * New: Make Vacuum request and handle response
      * Create: Save search results

4. Convert to SPA
  * Install Angular and modules
  * Convert Rails controller actions to JSON
