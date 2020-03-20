I was unmarshalling a soap request using the xml package, but after a couple of tests I realized  
the unmarshalled representation did not match the actual marshalled object.  
That’s a big issue if  you are marshalling/unmarshalling xml soap requests.  
  
If you are using simple xml documents it’s fine, but for more complex stuff you need to be aware.
For example try to unmarshal/marshal this and you will see what I’m talking about. (if it works now I would like to know)  
   
   
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:yourWebService">
```  

# Solution
Just a kludge, to bypass this issue I used marshall a simpler struct that comes in the soap request body,  
do whatever I need with that struct, the concatenate all in byte array.  

I just marshall this xml struct
<name>some_name</name>
<value>some_value</value>
into this  
  
```golang
type datasource_inputs struct {
  Name  string `xml:"name"`
  Value string `xml:"value"`
}
```
  
I read this great article when I was learning how to use the xml library  

https://astaxie.gitbooks.io/build-web-application-with-golang/en/07.1.html  

Here is the actual code :  

```golang
soapenv_head :=
[]byte(
`<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:yourWebService">
<soapenv:Header/>
<soapenv:Body>
<urn:callyourWebservice>`)

soapenv_close := []byte(
`</urn:callyourWebservice>
</soapenv:Body>
</soapenv:Envelope>`)

// Do something with the unmarshalled body

Uxml := append ( <your_xml_struct_that_has_an_array_of_the_appended_struct>, datasource_inputs {'name','value' }   )

output, err := xml.Marshal(Uxml)
   if err != nil {
    fmt.Printf("error: %v\n", err)
     panic(err)
   }
var buffer bytes.Buffer
buffer.Write(soapenv_head)
buffer.Write(output)
buffer.Write(soapenv_close)

fmt.Println(buffer.bytes() ) // this has your full soap request
```
