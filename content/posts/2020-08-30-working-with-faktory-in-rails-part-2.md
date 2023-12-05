+++
layout = 'post'
type = 'blog'
title = 'Golang:> Running jobs from faktory:> Part 2'
date = 2020-08-30T12:00:00+05:45
+++

In the last part, we pushed jobs in faktory server. Now we need to write a Job runner or worker to fetch jobs and run them. We will write it in Golang:

### Create Worker
If you haven't installed Golang in your machine then I suggest you go through the docs and install the latest version. Let's create a folder in our $GOPATH. *~/go/src/github.com/csvtojson/main.go*
The file will have the following code:


	package main

	import (
		"os"
		// this is the package required to create a client
	    worker "github.com/contribsys/faktory_worker_go" 
	)

	// workerFunc ...
	func workerFunc(ctx worker.Context, args ...interface{}) error {
		filePath := args[0].(string)
		convertToJSON(filePath)
		return nil
	}
   
	// main ...
	func main()  {
	   faktoryURL := "http://localhost:7419" // this is to be fetched from environment variable
	   os.Setenv("FAKTORY_PROVIDER", "MY_FAKTORY_URL")
	   os.Setenv("MY_FAKTORY_URL", faktoryURL)
	   mgr := worker.NewManager() // main job handler
   
	   // register job and function to execute them
	   mgr.Register("csv_to_json", workerFunc)
	   mgr.Concurrency = 20 // know your machine before setting this
	   mgr.ProcessStrictPriorityQueues("default") // only fetch from default queue from redis
   
	   // start processing jobs
	   mgr.Run()
	}

Now, let's write the function that will take in the path of csv file as an argument and start processing it and convert everything inside and write to a equivalent JSON file. I will write comments in code but will not go into detail of the program. If you understand Golang then this should be easy. Create another file with the same package name:
*~/go/src/github.com/csvtojson/converter.go*

	package main
	
	import(
		"fmt"
		"os"
		"encoding/csv"    // primary packages to process a csv
		"encoding/json"   // primary packages to process json
	)
	
	// this is the data we get from csv, every row contains these data
	type UserInfo struct { 
		Name string
		TxCount int
		TotalGain float
		TotalAsset float
		AssetName string
	}
	
	func convertToJSON(filePath *string) { 
		csvFile, err := os.Open(*filePath)
		
		if err != nil { 
			fmt.Println("Error while opening file:  ", err)
			return
		}
		defer CsvFile.Close()     // this ensures file is closed after the completion of the function
		
		reader := csv.NewReader(csvFile)    // method from the csv package we are using
		content, _ := reader.ReadAll()      // read and start storing everything in content variable
		
		if(len(content) < 1) {
			fmt.Println("Error while reading data: No Data Present")
			return 
		}
		
		// get all the headers from the csv file
		var allRecords []UserInfo
		var singleRecord UserInfo
		
		for _, row := range content { 
			singleRecord.Name = row[0]
			singleRecord.TxCount, _ = strconv.Atoi(row[1])
			singleRecord.TotalGain, _ = strconv.ParseFloat(row[2])
			singleRecord.TotalAsset, _ = strconv.ParseFloat(row[3])
			singleRecord.AssetName = row[4]
			allRecords = append(allRecords, singleRecord)
		}
		
		// convert the array into json
		jsonData, err := json.Marshal(allRecords)
		
		if err != nil { 
			fmt.Println("Error while marshal array to json")
			return
		}
		
		// create a jsonFile and insert the data
		jsonFile, err := os.Create(*filePath + ".json")
		
		if err != nil { 
			fmt.Println("Error while writing data to json file: ", err)
			return
		}
		
		defer jsonFile.Close()
		
		jsonFile.write(jsondata)
		jsonFile.Close()
	}
	
	
Let's build and run the program. From the root directory:

	go build
	./csvtojson
	
	
Now, this program can pull in jobs in the faktory server, process them. The best thing about this is we can build programs which can be fast, utilize concurrency. This binary is going to run as a single process which will consume faktory jobs. 

- Faktory is a standalone binary and needs a worker process to consume jobs(like the one we wrote above)
- Faktory uses RocksDB for embedded datastore internally to persist all job data, queues, errors etc whereas sidekiq uses redis, which is dumb. Sidekiq is written in ruby, to access data there is network call involved. 
- Sidekiq is limited to only one language, that's ruby, you can't execute jobs with golang, python or javascript. 
