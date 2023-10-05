安装扩展 `go get github.com/PuerkitoBio/goquery`

```go
package main

import (
	"fmt"
	"github.com/PuerkitoBio/goquery"
	file "github.com/alexchen/go_test/File"
	"log"
	"net/http"
	"strconv"
)

const PATH = "/Users/chentao/Downloads/book/"
const host = "https://www.qixiashu.com/yue/465/465479/"
// https://www.qixiashu.com/yue/62/62422/
func main() {
	begin := 939847
	var url string
	for begin < 949857 {
		url = host + strconv.Itoa(begin) + ".html"
		getContent(url)
		fmt.Println(url)
		begin++
	}
}

func getContent(url string) {
	res, err := http.Get(url)
	if err != nil {
		log.Fatal(err)
	}
	defer res.Body.Close()
	if res.StatusCode != 200 {
		log.Fatalf("status code error: %d %s", res.StatusCode, res.Status)
	}

	// Load the HTML document
	doc, err := goquery.NewDocumentFromReader(res.Body)
	if err != nil {
		log.Fatal(err)
	}

	title := doc.Find("h1").Text()
	fmt.Println("get " + title)
	bread, _ := doc.Find("div.contentbox").Html()
	file.AppendContent(PATH+title+".html", bread)
}

```