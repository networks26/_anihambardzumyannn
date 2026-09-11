# IP, TTL and Traceroute

In the previous lab, we looked at Ethernet.

An Ethernet frame contains, among other things, a source and destination Ethernet address.

But Ethernet is normally used to carry another protocol.

One of those protocols is IP.

An IPv4 packet contains a source IP address and a destination IP address.

Today we will look at what happens to an IP packet while it travels through a network.

---

## 1. What is your IP address?

On the server, run:

```sh
/sbin/ifconfig eth0
```

Find the IPv4 address of the server.

Write it here:

```text
IP address:
```

Also find the network mask.

```text
Network mask:
```

You have already seen Ethernet addresses before.

Find the Ethernet address of `eth0` too.

```text
Ethernet address:
```

Notice that the same interface has both:

* an Ethernet address
* an IP address

These addresses belong to different network layers and have different purposes.

---

## 2. Where does the server send packets?

Run:

```sh
/sbin/route -n
```

You should see the routing table.

Look for a line whose destination is:

```text
0.0.0.0
```

This is the **default route**.

What is the gateway for the default route?

Write it here:

```text
Default gateway:
```

The default gateway is the router to which the server sends packets when their destination is not on the local network.

For example, `8.8.8.8` is certainly not part of our local Ethernet network.

So when we send an IP packet to:

```text
8.8.8.8
```

the IP packet has:

```text
destination IP = 8.8.8.8
```

but the Ethernet frame carrying that packet is initially sent to our router.

The router then forwards the IP packet further.

And another router may forward it again.

And another one.

Let's try to see that happening.

---

# 3. TTL

An IPv4 packet contains a field called **TTL**:

```text
Time To Live
```

Despite its name, on modern IP networks you can think of TTL mainly as a **hop counter**.

Every router that forwards the packet decreases its TTL by one.

Imagine that a packet starts with:

```text
TTL = 3
```

The first router receives it and changes it to:

```text
TTL = 2
```

The second router changes it to:

```text
TTL = 1
```

The third router would have to change it to:

```text
TTL = 0
```

At that point, the router does **not** forward the packet.

Instead, it normally sends an ICMP message back to us saying that the TTL has expired.

Why do you think IP needs this mechanism?

Think about what could happen if routers accidentally formed a loop.

---

# 4. Let's deliberately make a packet expire

Normally `ping` chooses a reasonably large TTL.

But we can choose it ourselves.

Run:

```sh
ping -c 1 -t 1 8.8.8.8
```

`-c 1` means:

```text
send one packet
```

and:

```text
-t 1
```

sets:

```text
TTL = 1
```

Did `8.8.8.8` answer?

Probably not.

Instead, another machine should answer with a message similar to:

```text
Time to live exceeded
```

Write down the IP address of the machine that answered:

```text
TTL 1:
```

What machine do you think this is?

Compare it with the default gateway you found earlier.

---

## 5. Let the packet travel one router farther

Now run:

```sh
ping -c 1 -t 2 8.8.8.8
```

Write down the address that answered:

```text
TTL 2:
```

Now try:

```sh
ping -c 1 -t 3 8.8.8.8
```

```text
TTL 3:
```

Continue:

```sh
ping -c 1 -t 4 8.8.8.8
ping -c 1 -t 5 8.8.8.8
```

Write down what you see.

```text
TTL 4:

TTL 5:
```

You can continue with larger TTL values if you want.

At some point you may finally receive an answer from:

```text
8.8.8.8
```

---

# 6. You have almost invented a program

Suppose we wrote a program that did this automatically:

```text
send a packet with TTL = 1
remember who answers

send a packet with TTL = 2
remember who answers

send a packet with TTL = 3
remember who answers

send a packet with TTL = 4
remember who answers

...
```

What would that program show us?

It would show us the routers through which our packets travel.

Such a program already exists.

It is called:

```text
traceroute
```

---

# 7. Traceroute

Run:

```sh
traceroute 8.8.8.8
```

You may see something resembling:

```text
 1  ...
 2  ...
 3  ...
 4  ...
 5  ...
```

Each numbered line represents another hop along the path.

Compare this output with the addresses you discovered manually using different TTL values.

Do they correspond?

```text
What was hop 1?

What was hop 2?

What was hop 3?
```

You may also see:

```text
* * *
```

Does this necessarily mean that packets stopped there?

Look at whether later hops still answer.

If hop 6 says:

```text
* * *
```

but hop 7 answers, then packets clearly passed through hop 6.

It only means that we did not receive the expected answer from that router.

---

# 8. Try traceroute without host names

Run:

```sh
traceroute -n 8.8.8.8
```

Compare it with:

```sh
traceroute 8.8.8.8
```

What is different?

Without `-n`, traceroute may show names such as:

```text
something.example.net
```

With `-n`, it only shows IP addresses.

The translation between IP addresses and names is done by another system called **DNS**.

We will discuss DNS separately.

For now, remember:

```text
routing works with IP addresses
```

Host names are another layer of convenience on top of that.

---

# 9. Do the same experiment from Windows

Now use your Windows lab computer.

First find its IP configuration:

```cmd
ipconfig
```

Write down its IPv4 address:

```text
Windows IPv4 address:
```

Find its default gateway:

```text
Windows default gateway:
```

Now try a packet with TTL 1:

```cmd
ping -i 1 8.8.8.8
```

Then:

```cmd
ping -i 2 8.8.8.8
```

Then:

```cmd
ping -i 3 8.8.8.8
```

Again, you are discovering the route one hop at a time.

Windows has its own traceroute command.

It is called:

```cmd
tracert
```

Run:

```cmd
tracert 8.8.8.8
```

Compare this route with the route you saw from the server.

Are they identical?

Where do they differ?

```text
Server first hop:

Windows first hop:
```

---

# 10. Try another network

If you have a laptop, connect it to the university Wi-Fi, even to AUA_Guest then do the same test with the other.

Open a terminal or command prompt on your laptop.

On Windows:

```cmd
tracert 8.8.8.8
```

On Linux:

```sh
traceroute 8.8.8.8
```

On macOS:

```sh
traceroute 8.8.8.8
```

Compare this route with the route from the server.

They all have the same final destination:

```text
8.8.8.8
```

But do they take the same path?

Write down the first few hops from both networks.

```text
Server:

1.
2.
3.
4.

Lab Windows machine:

1.
2.
3.
4.



University Wi-Fi:

1.
2.
3.
4.
```

Where do the two paths become different?

Do they later appear to join the same network again?

---

# 11. Think about what you observed

Answer these questions.

### Question 1

When you send a packet to `8.8.8.8`, does your computer need to know the entire path to `8.8.8.8`?

Or does it only need to know where to send the packet **next**?

```text
Answer:
```

### Question 2

What happens to TTL when an IP packet passes through a router?

```text
Answer:
```

### Question 3

What happens when TTL reaches zero?

```text
Answer:
```

### Question 4

Why is TTL necessary?

What could happen without it if routers accidentally created a routing loop?

```text
Answer:
```

### Question 5

How can traceroute discover routers between you and a destination?

Explain it using TTL.

```text
Answer:
```

### Question 6

Why can two computers sending packets to the same destination take different routes?

For example:

```text
server -> 8.8.8.8

Lab machine -> 8.8.8.8

university Wi-Fi -> 8.8.8.8
```

```text
Answer:
```

---

# 12. One more thing: your address on the Internet

On the server, you found an IP address using:

```sh
/sbin/ifconfig eth0
```

Now run:

```sh
curl -4 https://ip.me
```

you can also try:

```sh
curl -4 https://api.ipify.org
echo
```

Compare the result with the address shown by:

```sh
/sbin/ifconfig eth0
```

Are they the same?

```text
Address on eth0:
```

```text
Address that the website detects
```

Now do

```
ipconfig /all
```

on Windows lab machine.

Also enter https://ip.me via Windows lab machine web browser.

Is the IP same?
Write down:

```text
Address on eth0:
```

```text
Address that the website detects
```



If they are different, something between this server and the Internet is translating addresses.

This is commonly called:

```text
NAT
```

or:

```text
Network Address Translation
```

We will return to NAT later.

For now, the important observation is simply this:

> The IP address configured on a machine is not necessarily the IP address that a distant Internet server sees.

---

# What we learned

An IP packet has a source and destination IP address.

Routers forward IP packets toward their destination.

Your computer does not need to know the complete route. Usually it only needs to know the appropriate **next hop**.

Every router decreases the IPv4 TTL.

When TTL reaches zero, the packet is discarded.

By deliberately sending packets with TTL values:

```text
1, 2, 3, 4, ...
```

we can discover successive routers along a path.

That is the basic idea behind:

```text
traceroute
```

and Windows:

```text
tracert
```

And the route to the same destination can be different depending on where the packet starts.


Bonus question:
Why operating systems limit TTL by fairly small numbers? For Linux and MacOS it is 64, for Windows it is 128.

Speculate below:


