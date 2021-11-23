# cn
cn lab questions

Q)3 nodes
ans:


#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/applications-module.h"
#include "ns3/traffic-control-module.h"
#include "ns3/flow-monitor-module.h"


using namespace ns3;


int main (int argc, char *argv[])
{
  double simulationTime = 10; //seconds
  
  std::string socketType = "ns3::UdpSocketFactory";
    

  NodeContainer nodes;
  nodes.Create (3);

  PointToPointHelper pointToPoint;
  pointToPoint.SetDeviceAttribute ("DataRate", StringValue ("10Mbps"));
  pointToPoint.SetChannelAttribute ("Delay", StringValue ("2ms"));
  pointToPoint.SetQueue ("ns3::DropTailQueue", "MaxSize", StringValue ("1p"));

  NetDeviceContainer devices01;
  devices01 = pointToPoint.Install (nodes.Get(0),nodes.Get(1));
  NetDeviceContainer devices12;
  devices12 = pointToPoint.Install (nodes.Get(1),nodes.Get(2));
  
  InternetStackHelper stack;
  stack.Install (nodes);

  Ipv4AddressHelper address;
  
  address.SetBase ("10.1.1.0", "255.255.255.0");
  Ipv4InterfaceContainer interfaces01 = address.Assign (devices01);
  
  address.SetBase ("10.1.2.0", "255.255.255.0");
  Ipv4InterfaceContainer interfaces12 = address.Assign (devices12);
  
  Ipv4GlobalRoutingHelper::PopulateRoutingTables();
  //Flow
  uint16_t port = 7;
  Address localAddress (InetSocketAddress (Ipv4Address::GetAny (), port));
  PacketSinkHelper packetSinkHelper (socketType, localAddress);
  ApplicationContainer sinkApp = packetSinkHelper.Install (nodes.Get (2));

  sinkApp.Start (Seconds (0.0));
  sinkApp.Stop (Seconds (simulationTime + 0.1));

  

  OnOffHelper onoff (socketType, Ipv4Address::GetAny ());
  onoff.SetAttribute ("OnTime",  StringValue ("ns3::ConstantRandomVariable[Constant=1]"));
  onoff.SetAttribute ("OffTime", StringValue ("ns3::ConstantRandomVariable[Constant=0]"));
  
  onoff.SetAttribute ("DataRate", StringValue ("50Mbps")); //bit/s
  ApplicationContainer apps;

  InetSocketAddress rmt (interfaces12.GetAddress (1), port);
  //rmt.SetTos (0xb8);
  AddressValue remoteAddress (rmt);
  onoff.SetAttribute ("Remote", remoteAddress);
  apps.Add (onoff.Install (nodes.Get (0)));
  apps.Start (Seconds (1.0));
  apps.Stop (Seconds (simulationTime + 0.1));

  FlowMonitorHelper flowmon;
  Ptr<FlowMonitor> monitor = flowmon.InstallAll();

  Simulator::Stop (Seconds (simulationTime + 5));
  Simulator::Run ();

  Ptr<Ipv4FlowClassifier> classifier = DynamicCast<Ipv4FlowClassifier> (flowmon.GetClassifier ());
  std::map<FlowId, FlowMonitor::FlowStats> stats = monitor->GetFlowStats ();
  std::cout << std::endl << "*** Flow monitor statistics ***" << std::endl;
  for(std::map<FlowId, FlowMonitor::FlowStats>::const_iterator iter = stats.begin(); iter!=stats.end();++iter){
  Ipv4FlowClassifier::FiveTuple t = classifier -> FindFlow (iter->first);
  std::cout << "Flow ID: " << iter->first << " Src Addr " << t.sourceAddress << " Dst Addr " << t.destinationAddress<< std::endl;  
  std::cout << "  Tx Packets/Bytes:   " << iter->second.txPackets << std::endl;
  std::cout << "  Tx Packets/Bytes:   " << iter->second.txPackets << std::endl;
  std::cout << "  Tx Packets/Bytes:   " << iter->second.txPackets << std::endl;
  std::cout << "  Throughput: " << iter->second.rxBytes * 8.0 / (iter->second.timeLastRxPacket.GetSeconds () -iter->second.timeFirstRxPacket.GetSeconds ()) / 1000000 << " Mbps" << std::endl;
  }
  Simulator::Destroy ();

  
  return 0;
}
  
  
  -----------------------------------------------------------------------------------------------------------------------
  Q) 4 nodes
  ans:
  

#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/applications-module.h"
#include "ns3/traffic-control-module.h"
#include "ns3/flow-monitor-module.h"


using namespace ns3;


int main (int argc, char *argv[])
{
  double simulationTime = 10; //seconds
  std::string socketType = "ns3::UdpSocketFactory";
    

  NodeContainer nodes;
  nodes.Create (4);

  PointToPointHelper pointToPoint;
  pointToPoint.SetDeviceAttribute ("DataRate", StringValue ("10Mbps"));
  pointToPoint.SetChannelAttribute ("Delay", StringValue ("2ms"));
  pointToPoint.SetQueue ("ns3::DropTailQueue", "MaxSize", StringValue ("1p"));

  NetDeviceContainer devices02;
  devices02 = pointToPoint.Install (nodes.Get(0),nodes.Get(2));
  NetDeviceContainer devices12;
  devices12 = pointToPoint.Install (nodes.Get(1),nodes.Get(2));
  NetDeviceContainer devices23;
  devices23 = pointToPoint.Install (nodes.Get(2),nodes.Get(3));

  InternetStackHelper stack;
  stack.Install (nodes);

  
  Ipv4AddressHelper address;
  
  address.SetBase ("10.1.1.0", "255.255.255.0");
  Ipv4InterfaceContainer interfaces02 = address.Assign (devices02);
  address.SetBase ("10.1.2.0", "255.255.255.0");
  Ipv4InterfaceContainer interfaces12 = address.Assign (devices12);
  address.SetBase ("10.1.3.0", "255.255.255.0");
  Ipv4InterfaceContainer interfaces23 = address.Assign (devices23);

  Ipv4GlobalRoutingHelper::PopulateRoutingTables();
  
  //UDP Flow
  uint16_t port = 7;
  Address localAddress (InetSocketAddress (Ipv4Address::GetAny (), port));
  PacketSinkHelper packetSinkHelper (socketType, localAddress);
  ApplicationContainer sinkApp = packetSinkHelper.Install (nodes.Get (3));

  sinkApp.Start (Seconds (0.0));
  sinkApp.Stop (Seconds (simulationTime + 0.1));



  OnOffHelper onoff (socketType, Ipv4Address::GetAny ());
  onoff.SetAttribute ("OnTime",  StringValue ("ns3::ConstantRandomVariable[Constant=1]"));
  onoff.SetAttribute ("OffTime", StringValue ("ns3::ConstantRandomVariable[Constant=0]"));
 
  onoff.SetAttribute ("DataRate", StringValue ("50Mbps")); //bit/s
  ApplicationContainer apps;

  InetSocketAddress rmt (interfaces23.GetAddress (1), port);
  //rmt.SetTos (0xb8);
  AddressValue remoteAddress (rmt);
  onoff.SetAttribute ("Remote", remoteAddress);
  apps.Add (onoff.Install (nodes.Get (0)));
  apps.Start (Seconds (1.0));
  apps.Stop (Seconds (simulationTime + 0.1));

  //TCP Flow
  uint16_t porttcp = 7;
  std::string socketTypetcp = "ns3::TcpSocketFactory";
  Address localAddresstcp (InetSocketAddress (Ipv4Address::GetAny (), porttcp));
  PacketSinkHelper packetSinkHelpert (socketTypetcp, localAddresstcp);
  ApplicationContainer sinkApptcp = packetSinkHelpert.Install (nodes.Get (3));

  sinkApptcp.Start (Seconds (0.0));
  sinkApptcp.Stop (Seconds (simulationTime + 0.1));



  OnOffHelper onofft (socketTypetcp, Ipv4Address::GetAny ());
  onofft.SetAttribute ("OnTime",  StringValue ("ns3::ConstantRandomVariable[Constant=1]"));
  onofft.SetAttribute ("OffTime", StringValue ("ns3::ConstantRandomVariable[Constant=0]"));
 
  onofft.SetAttribute ("DataRate", StringValue ("50Mbps")); //bit/s
  ApplicationContainer appstcp;

  InetSocketAddress rmtt (interfaces23.GetAddress (1), porttcp);
  //rmt.SetTos (0xb8);
  AddressValue remoteAddresst (rmtt);
  onofft.SetAttribute ("Remote", remoteAddresst);
  appstcp.Add (onofft.Install (nodes.Get (0)));
  appstcp.Start (Seconds (1.0));
  appstcp.Stop (Seconds (simulationTime + 0.1));



  FlowMonitorHelper flowmon;
  Ptr<FlowMonitor> monitor = flowmon.InstallAll();

  Simulator::Stop (Seconds (simulationTime + 5));
  Simulator::Run ();

  Ptr<Ipv4FlowClassifier> classifier = DynamicCast<Ipv4FlowClassifier> (flowmon.GetClassifier ());
  std::map<FlowId, FlowMonitor::FlowStats> stats = monitor->GetFlowStats ();
  std::cout << std::endl << "*** Flow monitor statistics ***" << std::endl;
  
  for(std::map<FlowId, FlowMonitor::FlowStats>::const_iterator iter = stats.begin(); iter!=stats.end();++iter){
  Ipv4FlowClassifier::FiveTuple t = classifier->FindFlow (iter->first);
  std::cout << "Flow ID: " << iter->first << " Src Addr " << t.sourceAddress << " Dst Addr " << 
t.destinationAddress<< std::endl; 
  
  
  std::cout << "  Tx Packets:   " << iter->second.txPackets<< std::endl;
  std::cout << "  Rx Packets:   " << iter->second.rxPackets<< std::endl;
  std::cout << "  lost Packets:   " << iter->second.lostPackets<< std::endl;
  
  std::cout << "  Throughput: " << iter->second.rxBytes * 8.0 /  
  (iter->second.timeLastRxPacket.GetSeconds () -  
  iter->second.timeFirstRxPacket.GetSeconds ()) / 1000000 << " Mbps" << std::endl;
  
}
  Simulator::Destroy ();

  return 0;
}
  
  
  
  ------------------------------------------------------------------------------------------------------------------------------------------------
  
  
  
  Q) extended or estimated service
  ans:
  
#include "ns3/core-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/network-module.h"
#include "ns3/applications-module.h"
#include "ns3/mobility-module.h"
#include "ns3/csma-module.h"
#include "ns3/internet-module.h"
#include "ns3/yans-wifi-helper.h"
#include "ns3/ssid.h"
#include "ns3/flow-monitor-module.h" 
#include "ns3/wifi-module.h" //imp to remebmmememmemmeeeeeeeeeeeeeeeeeeeeee

using namespace ns3;

NS_LOG_COMPONENT_DEFINE ("ThirdScriptExample");

int  main (int argc, char *argv[])
{
  
  uint32_t nCsma = 3;
  uint32_t nWifi = 3;
  double simulationTime = 10; //seconds 
  std::string socketType = "ns3::UdpSocketFactory"; 

  CommandLine cmd ;
  
  cmd.Parse (argc,argv);

  // The underlying restriction of 18 is due to the grid position
  // allocator's configuration; the grid layout will exceed the
  // bounding box if more than 18 nodes are provided.
  if (nWifi > 250 || nCsma >250)
    {
      std::cout << "nWifi should be 18 or less; otherwise grid layout exceeds the bounding box" << std::endl;
      return 1;
    }

  
  NodeContainer p2pNodes;
  p2pNodes.Create (2);

  PointToPointHelper pointToPoint;
  pointToPoint.SetDeviceAttribute ("DataRate", StringValue ("5Mbps"));
  pointToPoint.SetChannelAttribute ("Delay", StringValue ("2ms"));

  NetDeviceContainer p2pDevices;
  p2pDevices = pointToPoint.Install (p2pNodes);

  NodeContainer csmaNodes;
  csmaNodes.Add (p2pNodes.Get (1));
  csmaNodes.Create (nCsma);

  CsmaHelper csma;
  csma.SetChannelAttribute ("DataRate", StringValue ("100Mbps"));
  csma.SetChannelAttribute ("Delay", TimeValue (NanoSeconds (6560)));

  NetDeviceContainer csmaDevices;
  csmaDevices = csma.Install (csmaNodes);

  NodeContainer wifiStaNodes;
  wifiStaNodes.Create (nWifi);
  NodeContainer wifiApNode = p2pNodes.Get (0);

  YansWifiChannelHelper channel = YansWifiChannelHelper::Default ();
  YansWifiPhyHelper phy;
  phy.SetChannel (channel.Create ());

  WifiHelper wifi;
  wifi.SetRemoteStationManager ("ns3::AarfWifiManager");

  WifiMacHelper mac;
  Ssid ssid = Ssid ("ns-3-ssid");
  mac.SetType ("ns3::StaWifiMac",
               "Ssid", SsidValue (ssid),
               "ActiveProbing", BooleanValue (false));

  NetDeviceContainer staDevices;
  staDevices = wifi.Install (phy, mac, wifiStaNodes);

  mac.SetType ("ns3::ApWifiMac",
               "Ssid", SsidValue (ssid));

  NetDeviceContainer apDevices;
  apDevices = wifi.Install (phy, mac, wifiApNode);

  MobilityHelper mobility;

  mobility.SetPositionAllocator ("ns3::GridPositionAllocator",
                                 "MinX", DoubleValue (0.0),
                                 "MinY", DoubleValue (0.0),
                                 "DeltaX", DoubleValue (5.0),
                                 "DeltaY", DoubleValue (10.0),
                                 "GridWidth", UintegerValue (3),
                                 "LayoutType", StringValue ("RowFirst"));

  mobility.SetMobilityModel ("ns3::RandomWalk2dMobilityModel",
                             "Bounds", RectangleValue (Rectangle (-50, 50, -50, 50)));
  mobility.Install (wifiStaNodes);

  mobility.SetMobilityModel ("ns3::ConstantPositionMobilityModel");
  mobility.Install (wifiApNode);

  InternetStackHelper stack;
  stack.Install (csmaNodes);
  stack.Install (wifiApNode);
  stack.Install (wifiStaNodes);

  Ipv4AddressHelper address;

  address.SetBase ("10.1.1.0", "255.255.255.0");
  Ipv4InterfaceContainer p2pInterfaces;
  p2pInterfaces = address.Assign (p2pDevices);

  address.SetBase ("10.1.2.0", "255.255.255.0");
  Ipv4InterfaceContainer csmaInterfaces;
  csmaInterfaces = address.Assign (csmaDevices);

  address.SetBase ("10.1.3.0", "255.255.255.0");
  address.Assign (staDevices);
  address.Assign (apDevices);

 

  Ipv4GlobalRoutingHelper::PopulateRoutingTables ();
  
  //Flow
  uint16_t port = 7;
  Address localAddress (InetSocketAddress (Ipv4Address::GetAny (), port));
  PacketSinkHelper packetSinkHelper (socketType, localAddress);
  ApplicationContainer sinkApp = packetSinkHelper.Install (csmaNodes.Get (nCsma));

  sinkApp.Start (Seconds (0.0));
  sinkApp.Stop (Seconds (simulationTime + 0.1));

  uint32_t payloadSize = 1448;
  Config::SetDefault ("ns3::TcpSocket::SegmentSize", UintegerValue (payloadSize));

  OnOffHelper onoff (socketType, Ipv4Address::GetAny ());
  onoff.SetAttribute ("OnTime",  StringValue ("ns3::ConstantRandomVariable[Constant=1]"));
  onoff.SetAttribute ("OffTime", StringValue ("ns3::ConstantRandomVariable[Constant=0]"));
  onoff.SetAttribute ("PacketSize", UintegerValue (payloadSize));
  onoff.SetAttribute ("DataRate", StringValue ("50Mbps")); //bit/s
  ApplicationContainer apps;

  AddressValue remoteAddress (InetSocketAddress  (csmaInterfaces.GetAddress (nCsma), port));

  onoff.SetAttribute ("Remote", remoteAddress);
  apps.Add (onoff.Install (wifiStaNodes.Get(nWifi - 1)));
  
  apps.Start (Seconds (1.0));
  apps.Stop (Seconds (simulationTime + 0.1));


  Simulator::Stop (Seconds (10.0));

  FlowMonitorHelper flowmon;
  Ptr<FlowMonitor> monitor = flowmon.InstallAll();

  Simulator::Run ();
  monitor->CheckForLostPackets();
  
  
  Ptr<Ipv4FlowClassifier> classifier = DynamicCast<Ipv4FlowClassifier> (flowmon.GetClassifier ());
  std::map<FlowId, FlowMonitor::FlowStats> stats = monitor->GetFlowStats ();
  std::cout << std::endl << "*** Flow monitor statistics ***" << std::endl;
  for (std::map<FlowId, FlowMonitor::FlowStats>::const_iterator iter = stats.begin (); iter != stats.end 
(); ++iter) 
    { 
  Ipv4FlowClassifier::FiveTuple t = classifier->FindFlow (iter->first); 
      NS_LOG_UNCOND("Flow ID: " << iter->first << " Src Addr " << t.sourceAddress << " Dst Addr " << t.destinationAddress); 
      NS_LOG_UNCOND("Tx Packets = " << iter->second.txPackets); 
      std::cout << "Rx Packets   = " << iter->second.rxPackets<< std::endl;  
      std::cout << "Lost Packets = " << iter->second.lostPackets<< std::endl;  
      std::cout << "Throughput   = " << iter->second.rxBytes * 8.0 / 
(iter->second.timeLastRxPacket.GetSeconds()-iter->second.timeFirstTxPacket.GetSeconds()) / 1000000  
<< " Kbps"<< std::endl;  
    } 
  
  
  Simulator::Destroy ();
  return 0;
}
