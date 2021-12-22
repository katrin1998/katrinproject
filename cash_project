import json
import time
import urllib.parse
from http.server import HTTPServer
from http.server import BaseHTTPRequestHandler
import urllib


class CacheHandler(BaseHTTPRequestHandler):

    data = {}

    def write_headers(self): 
        self.send_response(200)
        self.send_header("Content-Type", "application/json")
        self.end_headers()

    def write_error(self, message):  
        err = {"error": message}
        self.wfile.write(bytes(json.dumps(err), "utf-8"))

    def do_POST(self): 
        self.write_headers()  
        content_len = int(self.headers.get('Content-Length'))  
        post_body = self.rfile.read(content_len) 
        if not post_body: 
            self.write_error("invalid request")
            return
        dataset = json.loads(post_body)  
        for key in ['date', 'data', 'ttl']:  
            if key not in dataset.keys(): 
                self.write_error(key + " not exists")
                return
        self.save_data(dataset['date'], dataset['data'], dataset['ttl'])  
        self.wfile.write(bytes(json.dumps({'success': True}), 'utf-8')) 

    def save_data(self, date, data, ttl): 
        if len(self.data) >= 30:  
            self.data = {}
        valid_till = time.time() + ttl  
        self.data[str(date)] = {'data': data, 'validTill': valid_till}  

    def do_GET(self):
        try:
            parsed = urllib.parse.urlparse(self.path)
            d = urllib.parse.parse_qs(parsed.query)
            self.write_headers()
            if 'date' not in d.keys():
                self.write_error("invalid request")
                return
            date = d["date"][0]
            if date in self.data.keys():
                data = self.data[date]
                if data['validTill'] <= time.time():
                    self.data.pop(date)
                    self.write_error("not found")
                    return
                self.wfile.write(bytes(json.dumps(self.data[date]['data']), "utf-8"))
            else:
                self.write_error("not found")
        except Exception:
            self.write_error("internal error")


def run(server_class=HTTPServer, handler_class=BaseHTTPRequestHandler):
    server_address = ('', 8080)
    httpd = server_class(server_address, handler_class)
    httpd.serve_forever()


if __name__ == '__main__':
    try:
        run(handler_class=CacheHandler)
    except KeyboardInterrupt:
        exit(0)
        
