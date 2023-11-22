# Wireshark

## Generating and Analyzing Traffic with RADIUS

Today I'll be analyzing traffic generated with RADIUS, a network authentication protocol that runs in the application layer.

If you've never heard of RADIUS, it's something you'd most likely find in the backend of company networks, managing access to resources like Wi-Fi and VPN services.

> RADIUS protocol itself does not fully encrypt its traffic. While it encrypts passwords, other information within a RADIUS packet is not encrypted, making it less secure than protocols that provide full packet encryption. This is why RADIUS is often used with additional security layers, like IPsec or TLS, especially when used over public networks.

We'll use the public RADIUS server to capture and analyze traffic. Specifically, we will test the authentication and analyze the encrypted data inside.

capture01.png

Since RADIUS operates on port 1812, let's set a filter in Wireshark to listen only on that port.

capture02.png

Open NTRadPing, the RADIUS client, which we will use to send an authentication request to the public server.

capture03.png

Examining the request, we see the username is visible and the password is encrypted.

capture04.png

Inside Wireshark, We can set RADIUS protocol preferences to decrypt the password:

capture05.png

We have now generated and analyzed RADIUS traffic.

## Basic HTTP Authentication

HTTP is, of course, the protocol through which servers and browsers communicate. It runs on port 80 and is unencrypted and not very secure. In a basic HTTP authentication, a server requests user ID and password from a client. We will use a public server that hosts an HTTP page.

capture06.png

As you see above, it is asking us for a username and a password and tell us the connection is not private, i.e. not secure. We will make two attempts: one with the wrong password, then one with the correct one. First, let's capture the traffic. We set our capture filter to port 80, where HTTP listens.

capture07.png

The first time, we intentionally input the wrong password, '1234'. The signin dialog reappears, prompting for the correct one. Let's input the right password: 'password.' We receive this JSON object:

capture08.png

Back in Wireshark, we see the first failed attempt, where we receive a 401 status code:

capture09.png

We can also observe on the second, correct attempt, we receive a 201 status, meaning accepted.

This is not secure, as we can plainly see everything that has happened, unlike with HTTPS.

Let's expand the first packet and drill down to HTTP and, below that, authorization. The base64 encoding of the password is there, which is insecure, as well as the original, incorrect password fully visible. We also have details of the client. In the successful request, similarly we see the correct password as well as the aforementioned details.

capture10.png
