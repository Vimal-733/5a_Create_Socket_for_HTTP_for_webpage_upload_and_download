# 5a_Create_Socket_for_HTTP_for_webpage_upload_and_download
## AIM :
To write a PYTHON program for socket for HTTP for web page upload and download
## Algorithm

1.Start the program.
<BR>
2.Get the frame size from the userk
<BR>
3.To create the frame based on the user request.
<BR>
4.To send frames to server from the client side.
<BR>
5.If your frames reach the server it will send ACK signal to client otherwise it will send NACK signal to client.
<BR>
6.Stop the program
<BR>
## Program 
# client 
```
import socket

def send_request(host, port, request, file_data=b""):
    """Send a raw HTTP request and return the response."""
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((host, port))
        s.sendall(request.encode() + file_data)
        response = s.recv(4096)
    return response


def upload_file(host, port, filename):
    """Upload a file to the server."""
    with open(filename, 'rb') as file:
        file_data = file.read()
        content_length = len(file_data)
        
    # Create HTTP POST request
    request = (
        f"POST /upload HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        f"Content-Length: {content_length}\r\n"
        f"Content-Type: application/octet-stream\r\n"
        f"\r\n"
    )
    
    response = send_request(host, port, request, file_data)
    print("Upload response:\n", response.decode(errors="ignore"))


def download_file(host, port, filename):
    """Download a file from the server."""
    request = f"GET /{filename} HTTP/1.1\r\nHost: {host}\r\n\r\n"
    response = send_request(host, port, request)
    
    # Split headers and content
    if b"\r\n\r\n" in response:
        header, file_content = response.split(b"\r\n\r\n", 1)
        with open("downloaded_" + filename, "wb") as f:
            f.write(file_content)
        print("Download successful! Saved as:", "downloaded_" + filename)
    else:
        print("Invalid response received.")


if __name__ == "__main__":
    host = 'localhost'  # Connect to local server
    port = 8080

    # Create a test file to upload
    with open('example.txt', 'w') as f:
        f.write("This is a sample file uploaded from client.\n")

    # Upload the file
    upload_file(host, port, 'example.txt')

    # Download the same file
    download_file(host, port, 'uploaded_file.txt')

```

# server

```
# server.py
import socket
import os

HOST = 'localhost'
PORT = 8080

def handle_client(conn):
    request = conn.recv(4096).decode(errors='ignore')
    print("Received request:\n", request)

    if request.startswith("POST /upload"):
        # Extract file data after headers
        if "\r\n\r\n" in request:
            header, file_data = request.split("\r\n\r\n", 1)
            with open("uploaded_file.txt", "wb") as f:
                f.write(file_data.encode())
            response = "HTTP/1.1 200 OK\r\n\r\nFile uploaded successfully"
        else:
            response = "HTTP/1.1 400 Bad Request\r\n\r\nMissing file data"
        conn.sendall(response.encode())

    elif request.startswith("GET /"):
        filename = request.split(" ")[1].lstrip("/")
        if os.path.exists(filename):
            with open(filename, "rb") as f:
                file_content = f.read()
            response = b"HTTP/1.1 200 OK\r\n\r\n" + file_content
            conn.sendall(response)
        else:
            conn.sendall(b"HTTP/1.1 404 Not Found\r\n\r\nFile not found")

    conn.close()

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen(1)
    print(f"Server running on {HOST}:{PORT}...")
    while True:
        conn, addr = s.accept()
        handle_client(conn)
```


## OUTPUT

<img width="1026" height="297" alt="image" src="https://github.com/user-attachments/assets/62f35a01-71f5-4c1b-84e7-5773fae0e23e" />

## Result
Thus the socket for HTTP for web page upload and download created and Executed
