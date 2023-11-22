# Wireshark

## Generating and Analyzing Traffic with RADIUS

Today I'll be analyzing traffic generated with RADIUS, a network authentication protocol that runs in the application layer.

If you've never heard of RADIUS, it's something you'd most likely find in the backend of company networks, managing access to resources like Wi-Fi and VPN services.

> RADIUS protocol itself does not fully encrypt its traffic. While it encrypts passwords, other information within a RADIUS packet is not encrypted, making it less secure than protocols that provide full packet encryption. This is why RADIUS is often used with additional security layers, like IPsec or TLS, especially when used over public networks.

We'll use the public RADIUS server to capture and analyze traffic. Specifically, we will test the authentication and analyze the encrypted data inside.

![Figure 1](/img/capture01.png 'Figure 1')

Since RADIUS operates on port 1812, let's set a filter in Wireshark to listen only on that port.

![Figure 2](/img/capture02.png 'Figure 2')

Open NTRadPing, the RADIUS client, which we will use to send an authentication request to the public server.

![Figure 3](/img/capture03.png 'Figure 3')

Examining the request, we see the username is visible and the password is encrypted.

![Figure 4](/img/capture04.png 'Figure 4')

Inside Wireshark, We can set RADIUS protocol preferences to decrypt the password:

![Figure 5](/img/capture05.png 'Figure 5')

We have now generated and analyzed RADIUS traffic.

## Basic HTTP Authentication

HTTP is, of course, the protocol through which servers and browsers communicate. It runs on port 80 and is unencrypted and not very secure. In a basic HTTP authentication, a server requests user ID and password from a client. We will use a public server that hosts an HTTP page.

![Figure 6](/img/capture06.png 'Figure 6')

As you see above, it is asking us for a username and a password and tell us the connection is not private, i.e. not secure. We will make two attempts: one with the wrong password, then one with the correct one. First, let's capture the traffic. We set our capture filter to port 80, where HTTP listens.

capture07.png

The first time, we intentionally input the wrong password, '1234'. The signin dialog reappears, prompting for the correct one. Let's input the right password: 'password.' We receive this JSON object:

![Figure 7](/img/capture07.png 'Figure 7')

Back in Wireshark, we see the first failed attempt, where we receive a 401 status code:

![Figure 8](/img/capture08.png 'Figure 8')

We can also observe on the second, correct attempt, we receive a 201 status, meaning accepted.

This is not secure, as we can plainly see everything that has happened, unlike with HTTPS.

Let's expand the first packet and drill down to HTTP and, below that, authorization. The base64 encoding of the password is there, which is insecure, as well as the original, incorrect password fully visible. We also have details of the client. In the successful request, similarly we see the correct password as well as the aforementioned details.

![Figure 10](/img/capture10.png 'Figure 10')

## HTTP Form-based Authentication and DNS

Let's capture and analyze another HTTP example and then let's also look at the DNS traffic.

Form-based authentication was created to address the problems of basic HTTP authentication. Form-based authentication uses the standard HTML form fields to pass the username and password values to the server. How does it do this?

Instead of sending a GET request along with the credentials, it sends a POST request and passes the credentials in the body of the request. We will use this site for testing:

![Figure 11](/img/capture11.png 'Figure 11')

Above, you see the site is not secure, as it uses HTTP.

Start capturing traffic in Wireshark filtered on port 80.

In testing site, enter credentials.

In the last example, we found the authentication inside the HTTP request. This time, we will find it in the HTML form fields.

![Figure 12](/img/capture12.png 'Figure 12')

But you may notice that now the username and password are plainly visible in cleartext - not even any base64 encoding this time. How is this more secure than HTTP?

The answer is that form-based authentication was designed to be used with HTTPS. Since, with HTTPS, the entire packet is encrypted, there's no need to hide the credentials.

On the flipside of this, this implies that entering your credentials in a form on a site that is not HTTPS is, needless to say, an absolutely no-go.

Now let's capture some DNS traffic. We set our Wireshark filter to port 53.

![Figure 13](/img/capture13.png 'Figure 13')

Is the above data encrypted? No. If it were, we would not be able to see the server whose DNS we need or the IP.
