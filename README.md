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
