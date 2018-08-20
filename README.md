# Creating a Multiuser AR Experience

# 创建一个多用户的AR体验

Transmit ARKit world-mapping data between nearby devices with the MultipeerConnectivity framework to create a shared basis for AR experiences.

通过MultipeerConnectivity框架在附近的设备之间传输ARKit世界地图来创造一个基础的多人共享AR体验

## Overview 总览

![Diagram showing AR experiences on two devices viewing, from two different perspectives, the same virtual red panda character sitting on a real table, after an ARWorldMap is transmitted from one device to the other.](Documentation/ConceptArt.png)

This sample app demonstrates a simple shared AR experience for two or more iOS 12 devices. Before exploring the code, try building and running the app to familiarize yourself with the user experience it demonstrates:

本示例程序演示了一个简单的多人共享AR场景,可以供两个或更多iOS 12设备之间共享.在浏览代码之前,先试着创建并运行一下这个app来熟悉其中的用户场景:

1. Run the app on one device. You can look around the local environment, and tap to place a virtual 3D character on real-world surfaces. (Tap again to place multiple copies of the character.)

   在一台设备上运行app.你可以环视周围环境,并点击屏幕,在真实世界的某个表面上放置一个虚拟的3D角色.(多次点击会复制多个.)

2. Run the app on a second device. On both device screens, a message indicates that they have automatically joined a shared session.

   在第二台设备上运行app.在两台设备的屏幕上,会出现同一条消息,提示你他们已经自动加入了一个共享的session中.

3. Tap the Send World Map button on one device. Make sure the other device is in an area that the first device visited before sending the map, or has a similar view of the surrounding environment.

   在第一台设备上,点击Send World Map按钮.发送地图之前,要确保其他的设备在第一台设备刚才的地区附近,或者周围环境场景类似.

4. The other device displays a message indicating that it has received the map and is attempting to use it. When that process succeeds, both devices show virtual content at the same real-world positions, and tapping on either device places virtual content visible to both.

   其他设备会显示一条消息:已收到地图并尝试使用它.处理成功后,所有的设备就在真实世界的相同位置显示虚拟内容,在任意一台设备上点击添加新虚拟物体,都可以在另外的设备上看到.

Follow the steps below to see how this app uses the [`ARWorldMap`][0] class to save and restore ARKit's spatial mapping state, and the [`MultipeerConnectivity`][1] framework to send world maps between nearby devices. 

按照下面的步骤来查看,本app是如何使用 [`ARWorldMap`][0] 类来保存及重建ARKit空间地图状态的,以及 [`MultipeerConnectivity`][1] 框架是如何在附近的设备之间发送世界地图的.

[0]:https://developer.apple.com/documentation/arkit/arworldmap
[1]:https://developer.apple.com/documentation/multipeerconnectivity

## Getting Started 开始

Requires Xcode 10.0, iOS 12.0 and two or more iOS devices with A9 or later processors.

要求10.0, iOS 12.0 并且两台以上带有A9或更新处理的iOS设备.

## Run the AR Session and Place AR Content 运行AR Session并放置AR内容

This app extends the basic workflow for building an ARKit app. (For details, see [Building Your First AR Experience][10].) It defines an [`ARWorldTrackingConfiguration`][11] with plane detection enabled, then runs that configuration in the [`ARSession`][12] attached to the [`ARSCNView`][13] that displays the AR experience.

这个app是对基础版ARKit app的扩展.(详情,见[Building Your First AR Experience][10])它定义了一个 [`ARWorldTrackingConfiguration`][11] 并启用了平面检测,然后在 [`ARSession`][12] 中使用这个配置项,并在 [`ARSCNView`][13] 中显示AR场景.

When [`UITapGestureRecognizer`][14] detects a tap on the screen, the [`handleSceneTap`](x-source-tag://PlaceCharacter) method uses ARKit hit-testing to find a 3D point on a real-world surface, then places an [`ARAnchor`][15] marking that position. When ARKit calls the delegate method [`renderer(_:didAdd:for:)`][16], the app loads a 3D model for [`ARSCNView`][17] to display at the anchor's position.

当 [`UITapGestureRecognizer`][14] 检测到屏幕点击事件, [`handleSceneTap`](x-source-tag://PlaceCharacter) 方法会使用ARKit的命中测试来在真实世界中找到一个3D点,然后放置一个 [`ARAnchor`][15] 来标志它的位置.当ARKit调用 [`renderer(_:didAdd:for:)`][16]代理方法时,app就加载一个3D模型并显示在锚点位置上.

[10]:https://developer.apple.com/documentation/arkit/building_your_first_ar_experience
[11]:https://developer.apple.com/documentation/arkit/arworldtrackingconfiguration
[12]:https://developer.apple.com/documentation/arkit/arsession
[13]:https://developer.apple.com/documentation/arkit/arscnview
[14]:https://developer.apple.com/documentation/uikit/uitapgesturerecognizer
[15]:https://developer.apple.com/documentation/arkit/aranchor
[16]:https://developer.apple.com/documentation/arkit/arscnview
[17]:https://developer.apple.com/documentation/arkit/arscnview

## Connect to Peer Devices 连接到对等网络中的设备

The sample [`MultipeerSession`](x-source-tag://MultipeerSession) class provides a simple abstraction around the [MultipeerConnectivity][20] features this app uses. After the main view controller creates a `MultipeerSession` instance (at app launch), it starts running an [`MCNearbyServiceAdvertiser`][21] to broadcast the device's ability to join multipeer sessions and an [`MCNearbyServiceBrowser`][22] to find other devices:

本示例中的 [`MultipeerSession`](x-source-tag://MultipeerSession) 类提供了一个对[MultipeerConnectivity][20] 的简单抽象.在主控制器中创建一个`MultipeerSession` 实例(在app启动时),它启动一个 [`MCNearbyServiceAdvertiser`][21] 来广播,以便加入multipeer sessions,还有一个 [`MCNearbyServiceBrowser`][22] 来发现其他设备:

``` swift
session = MCSession(peer: myPeerID, securityIdentity: nil, encryptionPreference: .required)
session.delegate = self

serviceAdvertiser = MCNearbyServiceAdvertiser(peer: myPeerID, discoveryInfo: nil, serviceType: MultipeerSession.serviceType)
serviceAdvertiser.delegate = self
serviceAdvertiser.startAdvertisingPeer()

serviceBrowser = MCNearbyServiceBrowser(peer: myPeerID, serviceType: MultipeerSession.serviceType)
serviceBrowser.delegate = self
serviceBrowser.startBrowsingForPeers()
```
[View in Source](x-source-tag://MultipeerSetup)

When the [`MCNearbyServiceBrowser`][22] finds another device, it calls the [`browser(_:foundPeer:withDiscoveryInfo:)`][23] delegate method. To invite that other device to a shared session, call the browser's [`invitePeer(_:to:withContext:timeout:)`][24] method:

当 [`MCNearbyServiceBrowser`][22] 找到其他设备后,会调用[`browser(_:foundPeer:withDiscoveryInfo:)`][23] 代理方法.要邀请其他设备加入一个共享session中,需要调用brower的[`invitePeer(_:to:withContext:timeout:)`][24] 方法:

``` swift
public func browser(_ browser: MCNearbyServiceBrowser, foundPeer peerID: MCPeerID, withDiscoveryInfo info: [String: String]?) {
    // Invite the new peer to the session.
    // 邀请新的网络成员加入session.
    browser.invitePeer(peerID, to: session, withContext: nil, timeout: 10)
}
```
[View in Source](x-source-tag://FoundPeer)

When the other device receives that invitation, [`MCNearbyServiceAdvertiser`][21] calls the [`advertiser(_: didReceiveInvitationFromPeer:withContext:invitationHandler:`][25] delegate method. To accept the invitation, call the provided `invitationHandler`:

当其他设备收到邀请后, [`MCNearbyServiceAdvertiser`][21] 会调用[`advertiser(_: didReceiveInvitationFromPeer:withContext:invitationHandler:`][25] 代理方法.要接受邀请,调用提供的 `invitationHandler`:

``` swift
func advertiser(_ advertiser: MCNearbyServiceAdvertiser, didReceiveInvitationFromPeer peerID: MCPeerID, withContext context: Data?, invitationHandler: @escaping (Bool, MCSession?) -> Void) {
    // Call handler to accept invitation and join the session.
    // 调用回调,接受邀请并加入session.
    invitationHandler(true, self.session)
}
```
[View in Source](x-source-tag://AcceptInvite)

- Important: This app automatically joins the first nearby session it finds. Depending on the kind of shared AR experience you want to create, you may want to more precisely control broadcasting, invitation, and acceptance behavior. See the [MultipeerConnectivity][20] documentation for details.

  重要提示:本app会自动加入附近发现的第一个session.你可能想要创建不同类型的AR体验,这样就需要更精细地控制广播,邀请及接受行为.详细内容请看 [MultipeerConnectivity][20] 文档.

In a multipeer session, all participants are by definition equal peers; there is no explicit separation of devices into host (or server or master) and guest (or client or slave) roles. However, you may wish to define such roles for your own AR experience. For example, a multiplayer game design might require a server role to arbitrate gameplay. If you need to separate peers by role, you can choose a way to do so that fits the design of your app. For example:

在一个multipeer session中,所有的参与者是平等的;没有明确的设备来充当主机或服务器的角色,也没有从机或客户端的角色.但是,你可能想要为你自己的AR场景定义这样的角色.例如,一个多人游戏的设计可能要求一个服务器的角色来仲裁游戏设置.如果你需要单独的不同的角色,你可以选择一种方式来设计你的app.例如:

- Have the user choose whether to act as a host or guest before starting a session. The host uses [`MCNearbyServiceAdvertiser`][21] to broadcast availability, and guests use [`MCNearbyServiceBrowser`][22] to find a host to join.

  在session开始之前,有一个选择主机或从机角色的选项.主机使用 [`MCNearbyServiceAdvertiser`][21] 来广播,从机使用[`MCNearbyServiceBrowser`][22] 来发现主机并加入.

- Join a session as peers, then negotiate between peers to nominate a master. (This approach can be helpful for designs that need a master role but also allow peers to join or leave at any time.)

  作为对等成员加入session后,选举一个成员做为主机.(这个方法在一些情况下非常有用,比如,游戏需要一个主机角色,但同时又允许成员随时加入或离开.)


[20]:https://developer.apple.com/documentation/multipeerconnectivity
[21]:https://developer.apple.com/documentation/multipeerconnectivity/mcnearbyserviceadvertiser
[22]:https://developer.apple.com/documentation/multipeerconnectivity/mcnearbyservicebrowser
[23]:https://developer.apple.com/documentation/multipeerconnectivity/mcnearbyservicebrowserdelegate/1406926-browser
[24]:https://developer.apple.com/documentation/multipeerconnectivity/mcnearbyservicebrowser/1406944-invitepeer
[25]:https://developer.apple.com/documentation/multipeerconnectivity/mcnearbyserviceadvertiserdelegate/1406971-advertiser


## Capture and Send the AR World Map 捕捉并发送AR世界地图

An [`ARWorldMap`][0] object contains a snapshot of all the spatial mapping information that ARKit uses to locate the user's device in real-world space. Reliably sharing a map to another device requires two key steps: finding a good time to capture a map, and capturing and sending it.

一个[`ARWorldMap`][0] 对象包含了一个所有空间地图信息的快照,ARKit使用这些信息来在真实世界空间中定位用户的设备.向另一台设备共享地图需要两个关键性步骤:找准时机来捕捉一个地图,然后捕捉并发送出去.

ARKit provides a [`worldMappingStatus`][30] value that indicates whether it's currently a good time to capture a world map (or if it's better to wait until ARKit has mapped more of the local environment). This app uses that value to provide visual feedback on its Send World Map button:

ARKit提供了一个 [`worldMappingStatus`][30] 值来标志当前是否是捕捉世界地图的合适时机(或者等到ARKit捕捉到足够的本地环境信息).本app用这个值来在Send World Map按钮上提供反馈信息:

``` swift
switch frame.worldMappingStatus {
case .notAvailable, .limited:
    sendMapButton.isEnabled = false
case .extending:
    sendMapButton.isEnabled = !multipeerSession.connectedPeers.isEmpty
case .mapped:
    sendMapButton.isEnabled = !multipeerSession.connectedPeers.isEmpty
}
mappingStatusLabel.text = frame.worldMappingStatus.description
```
[View in Source](x-source-tag://CheckMappingStatus)

When the user taps the Send World Map button, the app calls [`getCurrentWorldMap`][31] to capture the map from the running ARSession, then serializes it to a [`Data`][32] object with [`NSKeyedArchiver`][33] and sends it to other devices in the multipeer session:

当用户点击Send World Map按钮,app调用 [`getCurrentWorldMap`][31] 从当前运行中的ARSession中捕捉一个地图,然后用 [`NSKeyedArchiver`][33] 序列化为 [`Data`][32] 对象,然后在multipeer session中发送给其他设备:

``` swift
sceneView.session.getCurrentWorldMap { worldMap, error in
    guard let map = worldMap
        else { print("Error: \(error!.localizedDescription)"); return }
    guard let data = try? NSKeyedArchiver.archivedData(withRootObject: map, requiringSecureCoding: true)
        else { fatalError("can't encode map") }
    self.multipeerSession.sendToAllPeers(data)
}
```
[View in Source](x-source-tag://ReceiveData)

[30]:https://developer.apple.com/documentation/arkit/arframe/2990930-worldmappingstatus
[31]:https://developer.apple.com/documentation/arkit/arsession/2968206-getcurrentworldmap
[32]:https://developer.apple.com/documentation/foundation/data
[33]:https://developer.apple.com/documentation/foundation/nskeyedarchiver

## Receive and Relocalize to the Shared Map 接收并重定位到共享的地图中

When a device receives data sent by another participant in the multipeer session, the [`session(_:didReceive:fromPeer:)`][40] delegate method provides that data. To make use of it, the app uses [`NSKeyedUnarchiver`][41] to deserialize an [`ARWorldMap`][0] object, then creates and runs a new [`ARWorldTrackingConfiguration`][11] using that map as the [`initialWorldMap`][42]:

当一个设备从multipeer session中的其他成员接收到数据后,[`session(_:didReceive:fromPeer:)`][40] 代理方法中会拿到这个数据.要使用它,app使用 [`NSKeyedUnarchiver`][41] 来反序列化出一个 [`ARWorldMap`][0] 对象,然后创建并运行一个新的[`ARWorldTrackingConfiguration`][11] ,将反序列化出的地图作为 [`initialWorldMap`][42]:

``` swift
if let worldMap = try NSKeyedUnarchiver.unarchivedObject(ofClass: ARWorldMap.self, from: data) {
    
    // Run the session with the received world map.
    // 从接收到的世界地图中启动session.
    let configuration = ARWorldTrackingConfiguration()
    configuration.planeDetection = .horizontal
    configuration.initialWorldMap = worldMap
    sceneView.session.run(configuration, options: [.resetTracking, .removeExistingAnchors])
    
    // Remember who provided the map for showing UI feedback.
    // 保存地图提供者,以显示在UI上.
    mapProvider = peer
}
```
[View in Source](x-source-tag://UnarchiveWorldMap)

ARKit then attempts to *relocalize* to the new world map—that is, to reconcile the received spatial-mapping information with what it senses of the local environment. For best results: 

ARKit接着会尝试去*重定位relocalize*到新的世界地图——也就是,使接收到的空间地图信息与本地感知到的环境信息相匹配.要得到最好的结果:

1. Thoroughly scan the local environment on the sending device before sharing a world map.

   发送设备在共享地图前,要尽可能认真彻底地扫瞄本地环境.

2. Place the receiving device next to the sending device, so that both see the same view of the environment.

   将接收设备放置在发送设备旁边,这样两者就能看到同样的环境.

[40]:https://developer.apple.com/documentation/multipeerconnectivity/mcsessiondelegate/1406934-session
[41]:https://developer.apple.com/documentation/foundation/nskeyedunarchiver
[42]:https://developer.apple.com/documentation/arkit/arworldtrackingconfiguration/2968180-initialworldmap

## Share AR Content and User Actions 共享AR内容和用户动作

Sharing the world map also shares all existing anchors. In this app, this means that as soon as a receiving device relocalizes to the world map, it shows all the 3D characters that were placed by the sending device before it captured and sent a world map. However, recording and transmitting a world map and relocalizing to a world map are time-consuming, bandwidth-intensive operations, so you should take those steps only once, when a new device joins a session. 

共享世界地图也会共享所有已经存在的锚点.在本app中,这就意味着,只要以接收设备重定位到世界地图中,它会展示发送方放置的所有3D特征.但是,录制和广播世界地图,以及重定位到世界地图中是需要消耗时间的,对带宽非常敏感的操作,所以你应该在新设备加入session时只进行一次操作.

To create an ongoing shared AR experience, where each user's actions affect the AR scene visible to other users, after each device relocalizes to the same world map you should share only the information needed to recreate each user action. For example, in this app the user can tap to place a virtual 3D character in the scene. That character is static, so all that is needed to place the character on another participating device is the character's position and orientation in world space.

要创建一个不断变化的共享AR场景,当任一个用户有所动作时就可以影响AR场景中的其他用户,那么你应该在其他设备重定位到相同的世界地图之后,只共享那些需要在各个用户设备上重新创建的动作.例如,在本app中,用户可以点击屏幕在场景中放置3D物体.这个物体是静止不动的,所以其他参与者的设备要展示这个物体,只需要它在世界坐标空间的位置和朝向信息就够了.

This app communicates virtual character positions by sharing [`ARAnchor`][15] objects between peers. When one user taps in the scene, the app creates an anchor and adds it to the local [`ARSession`][12], then serializes that [`ARAnchor`][15] using [`NSKeyedArchiver`][32] and sends it to other devices in the multipeer session:

本app同步虚拟角色位置是通过在网络成员间共享 [`ARAnchor`][15] 对象实现的.当一个用户点击屏幕,app会创建一个锚点,并将其添加到本地的 [`ARSession`][12]中,然后用 [`NSKeyedArchiver`][32]将 [`ARAnchor`][15] 序列化,并发送给multipeer session中的其他设备:

``` swift
// Place an anchor for a virtual character. The model appears in renderer(_:didAdd:for:).
// 为虚拟角色放置一个锚点.模型会在renderer(_:didAdd:for:)方法中出现.
let anchor = ARAnchor(name: "panda", transform: hitTestResult.worldTransform)
sceneView.session.add(anchor: anchor)

// Send the anchor info to peers, so they can place the same content.
// 将锚点信息发送给网络成员,以便其放置相同的内容.
guard let data = try? NSKeyedArchiver.archivedData(withRootObject: anchor, requiringSecureCoding: true)
    else { fatalError("can't encode anchor") }
self.multipeerSession.sendToAllPeers(data)
```
[View in Source](x-source-tag://PlaceCharacter)

When other peers receive data from the multipeer session, they test for whether that data contains an archived [`ARAnchor`][15]; if so, they decode it and add it to their session:

当其他网络成员从multipeer session中接收到数据,他们测试一下该数据是否包含一个归档的 [`ARAnchor`][15];如果是,则将其解码并添加到他们的session中:

``` swift
if let anchor = try NSKeyedUnarchiver.unarchivedObject(ofClass: ARAnchor.self, from: data) {
    
    sceneView.session.add(anchor: anchor)
}
```
[View in Source](x-source-tag://ReceiveData)

This is just one strategy for adding dynamic features to a shared AR experience—many other strategies are possible. Choose one that fits the user interaction, rendering, and networking requirements of your app. For example, a game where users throw projectiles in the AR world space might define custom data types with attributes like initial position and velocity, then use Swift's [`Codable`][50] protocols to serialize that information to a binary representation for sending over the network. 

这只是添加动态特征到共享AR体验的一种策略——还有其他方法也是可行的.挑选一个适合你的app中的用户交互,渲染和网络要求的方案.例如,一个让玩家在AR世界中互相扔投射物的游戏,可能会定义一些自定义的数据类型,比如初始化位置和速度,然后使用Swift的 [`Codable`][50] 协议来序列化这些信息到二进制数据中并通过网络发送出去.



[50]:https://developer.apple.com/documentation/swift/codable
