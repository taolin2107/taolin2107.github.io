---
title: 用Socket实现TCP，UDP通信
date: 2015-02-16 16:37:41
tags: [java, tcp, udp, 通信协议]
categories: java
---

## 1. 用Socket实现UDP通信

UDP适用于一次传送少量数据、对可靠性要求不高的应用环境。

```java
public class UDPServer {

    public static void main(String[] args) {
        try {
            DatagramSocket ds = new DatagramSocket();
            byte[] buffer = "udp test".getBytes();
            DatagramPacket dp = new DatagramPacket(buffer, buffer.length, InetAddress.getByName("localhost"), 8000);
            // 将localhost换为255.255.255.255可以发送广播，对该局域网内所有机器发送消息
            ds.send(dp);
            ds.close();
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}


public class UDPClient {

    public static void main(String[] args) {
        try {
            DatagramSocket ds = new DatagramSocket(8000);
            byte[] buffer = new byte[1024];
            DatagramPacket dp = new DatagramPacket(buffer, buffer.length);
            ds.receive(dp);
            String ip = dp.getAddress().getHostAddress();
            String data = new String(dp.getData(), 0, dp.getLength());
            int port = dp.getPort();
            System.out.println("ip: " + ip + "; data: " + data + "; port: " + port);
            ds.close();
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
<!-- more -->


## 2. 用Socket实现TCP通信

TCP因为要经过三次“对话”，传输可靠，适用于传输大文件，传输速度比UDP慢。

```java
public class TCPServer {

    public static void main(String[] args) {
        try {
            ServerSocket server = new ServerSocket(8001);
            boolean flag = true;
            while (flag) {
                final Socket client = server.accept();
                System.out.println("Connect to client " + client + " remote add: " + client.getRemoteSocketAddress());
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            PrintStream ps = new PrintStream(client.getOutputStream());
                            BufferedReader br = new BufferedReader(new InputStreamReader(client.getInputStream()));
                            while (true) {
                                String line = br.readLine();
                                System.out.println("client: " + line);
                                ps.println(line + " copied");
                                if ("bye".equals(line)) {
                                    break;
                                }
                            }
                            ps.close();
                            br.close();
                            client.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

public class TCPClient {

    public static void main(String[] args) {
        try {
            Socket client = new Socket("localhost", 8001);
            client.setSoTimeout(10000);
            BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
            PrintStream out = new PrintStream(client.getOutputStream());
            BufferedReader br = new BufferedReader(new InputStreamReader(client.getInputStream()));
            while (true) {
                System.out.print("input: ");
                String str = input.readLine();
                out.println(str);
                if ("bye".equals(str)) {
                    break;
                }
                String read = br.readLine();
                System.out.println("server: " + read);
            }
            input.close();
            client.close();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```
