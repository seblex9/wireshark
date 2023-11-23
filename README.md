# Wireshark Network Traffic Analysis

## Table of Contents

- [Introduction](#introduction)
- [Generating and Analyzing Traffic with RADIUS](#generating-and-analyzing-traffic-with-radius)
- [Basic HTTP Authentication](#basic-http-authentication)
- [HTTP Form-based Authentication and DNS](#http-form-based-authentication-and-dns)
- [Capturing and Analyzing a Telnet Session](#capturing-and-analyzing-a-telnet-session)
- [Capturing and Analyzing SSH Sessions](#capturing-and-analyzing-ssh-sessions)
- [Basic HTTP Authentication](#basic-http-authentication)
- [Generate, Capture, Analyze, and Decrypt HTTPS Traffic](#generate-capture-analyze-and-decrypt-https-traffic)
- [Basic HTTP Authentication](#basic-http-authentication)
- [Contact](#contact)

## Introduction:

This research project examines network traffic generated via different protocols using Wireshark. The study aims to provide comprehensive insights into the inner workings of network communication, focusing on authentication, encryption, and security protocols.

## Generating and Analyzing Traffic with RADIUS

Today, I'll be analyzing traffic generated with RADIUS, a network authentication protocol that runs in the application layer.

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

Inside Wireshark, we can set RADIUS protocol preferences to decrypt the password:

![Figure 5](/img/capture05.png 'Figure 5')

We have now generated and analyzed RADIUS traffic.

[Back to Top](#table-of-contents)

## Basic HTTP Authentication

HTTP is, of course, the protocol through which servers and browsers communicate. It runs on port 80 and is unencrypted and not very secure. In a basic HTTP authentication, a server requests user ID and password from a client. We will use a public server that hosts an HTTP page.

![Figure 6](/img/capture06.png 'Figure 6')

As you see above, it is asking us for a username and a password and tell us the connection is not private, i.e. not secure. We will make two attempts: one with the wrong password, then one with the correct one. First, let's capture the traffic. We set our capture filter to port 80, where HTTP listens.

![Figure 7](/img/capture07.png 'Figure 7')

The first time, we intentionally input the wrong password, '1234'. The signin dialog reappears, prompting for the correct one. Let's input the right password: 'password.' We receive this JSON object:

![Figure 8](/img/capture08.png 'Figure 8')

Back in Wireshark, we see the first failed attempt, where we receive a 401 status code:

![Figure 9](/img/capture09.png 'Figure 9')

We can also observe on the second, correct attempt, we receive a 201 status, meaning accepted.

This is not secure, as we can plainly see everything that has happened, unlike with HTTPS.

Let's expand the first packet and drill down to HTTP and, below that, authorization. The base64 encoding of the password is there, which is insecure, as well as the original, incorrect password fully visible. We also have details of the client. In the successful request, similarly we see the correct password as well as the aforementioned details.

![Figure 10](/img/capture10.png 'Figure 10')

[Back to Top](#table-of-contents)

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

> Conversely, this implies that entering your credentials in a form on a site that is not HTTPS is, needless to say, an absolutely no-go.

Now let's capture some DNS traffic. We set our Wireshark filter to port 53.

![Figure 13](/img/capture13.png 'Figure 13')

Is the above data encrypted? No. If it were, we would not be able to see the server whose DNS we need or the IP.

[Back to Top](#table-of-contents)

## Capturing and Analyzing a Telnet Session

telnet is a protocol that was built to access and manage devices remotely. its secure equivalent is SSH. here we will telnet to tty.sdf.org. Let's create an account first:

![Figure 14](/img/capture14.png 'Figure 14')

Telnet operates on port TCP 23. In Wireshark, we set a capture filter on port 23. We telnet to the server using Windows Powershell.

![Figure 15](/img/capture15.png 'Figure 15')

Note that telnet is not secure at all. If we right-click on the first packet in Wireshark, go to Follow and choose TCP Stream, you will see the exact output you saw earlier when we first logged in to our telnet session.

![Figure 16](/img/capture16.png 'Figure 16')

One thing to notice here is that my login name, 'seblex9', has every letter doubled, with the letters alternating between red and blue:

![Figure 17](/img/capture17.png 'Figure 17')

The red represents what we send to the server and the blue is what the server sends us back.

What about the password? You'll notice that's only in red. Well, what telnet does is it echoes back to us everything we are supposed to see on our screen. But you'll recall when inputting a password in Unix based systems, you do not actually see what you're typing. In other words, the server never send our password back to us, so the letters are red to represent what we sent.

[Back to Top](#table-of-contents)

## Capturing and Analyzing SSH Sessions

So far we have looked at unencrypted sources of traffic. Now let's look at encrypted, this means text is encrypted with an algorithm and a key, resulting in cyphertext which can only be viewed in its original form if it's decrypted with the correct key. This is the simplest form of encryption.

This is brings to SSH. SSH is used for the same purpose

as telnet, to access and manage devices remotely. Unlike telnet, it is secure. SSH uses port 22.

SSH uses the security protocol CLS, same one that HTTPS uses. You may have heard of SSL. This is the deprecated version of CLS.

We will generate our SSH traffic the same way with did with telnet, via the powershell.

This time, instead of using a capture filter to only catch traffic on port 22, let's just capture all the traffic that comes through. We'll do this with the following filter:

![Figure 18](/img/capture18.png 'Figure 18')

Now let's do something interesting. We will first login to the server with telnet, log out, and then log in again with SSH. For purposes of brevity, screenshots for this are ommitted. Next, let's look in Wireshark. Can Wireshark tell that we in effect had two different "conversations"?

In Wireshark, we can go to Statistics, then Conversations. Since both SSH and telnet use TCP, let's click on the TCP tab. And here we see the TCP tab shows us we had two conversations. Also, in the Port column, we see the ports for our telnet and SSH sessions, shown by ports 23 and 22, respectively.

![Figure 19](/img/capture19.png 'Figure 19')

If we click on the row for our telnet session and click 'follow stream' on the bottom, we once again see the entire unencrypted output, same as we got in the telnet session above.

Doing the same with the SSH session, we see everything is encrypted:

![Figure 20](/img/capture20.png 'Figure 20')

[Back to Top](#table-of-contents)

## Generate, Capture, Analyze, and Decrypt HTTPS Traffic

HTTPS is HTTP over TLS. It operates on port 443.

In Wireshark, we set a capture filter on port 443.

After generating some random traffic and inspecting a packet, we see under the TLS section that we're using http-over-tls protocol, i.e. HTTPS. We also see all the information has been encrypted.

![Figure 21](/img/capture21.png 'Figure 21')

How would we decrypt this traffic in Wireshark? We will use a premaster secret key. Our browser, which is the client, will generate this premaster secret key and we will log it in an SSL key log file. The web server will then use this key to generate a master secret key that encrypts all traffic. Since we have our premaster key in the log file, we can pass it to Wireshark to decrypt the data.

First, we set an environment variable in Windows so that we can instruct the browser to save this premaster secret key in a log file.

![Figure 22](/img/capture22.png 'Figure 22')

We now tell Wireshark to use our saved log file, ssh-keys.log by adding the log file path in the TLS protocol found under preferences:

![Figure 23](/img/capture23.png 'Figure 23')

Finally, let's capture the HTTPS traffic and test it. In Wireshark, we set a capture filter on port 443. In our browser, navigate to this page:

![Figure 24](/img/capture24.png 'Figure 24')

Back in Wireshark, we once again go to Statistics > Conversations.

![Figure 25](/img/capture25.png 'Figure 25')

There are quite a few conversations in there, so let's send a ping the server we connected to in order to identify our conversation with Simple Site above. In Windows command prompt:

![Figure 26](/img/capture26.png 'Figure 26')

We see the IP address begins with 185.199.

Back in Wireshark, we look for the IP address.

![Figure 27](/img/capture27.png 'Figure 27')

Click on it and choose 'follow stream':

![Figure 28](/img/capture28.png 'Figure 28')

And as you see, everything has been unencrypted.

[Back to Top](#table-of-contents)

## Contact

For further inquiries, professional networking, or in-depth discussions on network security and protocol analysis, please don't hesitate to connect with me on LinkedIn at [linkedin.com/in/seblex/](linkedin.com/in/seblex).

[Back to Top](#table-of-contents)
