---
layout: post
title: Extending Hybrid Lab Workflow Virtual Hosts
date: '2012-11-01 22:08:00'
tags:
- build
- labmanagement
---

In an [earlier post](http://colinsalmcorner.blogspot.com/2012/10/hybrid-lab-workflow-standard-lab.html) I talked about my [Hybrid Lab Workflow](http://hybridlabworkflow.codeplex.com/) – this workflow allows you to do a Build-Deploy-Test workflow against a TFS 2012 Standard Environment, and as long as the environment is composed of VMs and you’re able to connect to the VM Host, then you can apply pre-deployment snapshots and take post-deployment snapshots. I also [blogged](http://colinsalmcorner.blogspot.com/2012/11/developing-hybrid-lab-workflow.html) about the “nastiness” of PsKill and PsExec for getting the Lab into a workable state after snapshots were applied. In this post I’ll talk about how you could use exactly the same workflow for another Virtual Host – say VMWare or something else.

## The Magic – MEF

When I first thought of this workflow, I immediately thought that I could make the Virtual machine and Virtual Hosts interfaces in the workflow, so that you could hook into any virtualization platform that you want. Lab Management itself if oblivious to the virtualization platform you use (well, in Standard Environments anyway) since it treats the machines as if they are physical (not through the Virtual Host, which is what happens in SCVMM Environments).

So I created an IVirtualHost interface and an IVirtualMachine interface. I then created an enumeration of VirtualHostTypes and a VirtualHostFactory that could instantiate an IVirtualHost concrete class based on the enumeration you fed in. I soon realised that if you wanted to add a Host Type, you’d need to recompile the factory and update the workflow – it would be a bit messy. I ideally wanted something a little more “dynamic”. So I thought of the [Managed Extensibility Framework (MEF)](http://msdn.microsoft.com/en-us/library/dd460648.aspx) that is now baked into .NET. It’s designed exactly for this sort of problem.

I created a VirtualHostContainer that could dynamically load assemblies and find IVirtualHost implementations. Then I created a static class that uses the container to enumerate the HostTypes available and get you one when you need it.

## VirtualHost Container

    namespace HybridLab.Virtual.Interfaces<br>{<br> internal class VirtualHostContainer<br> {<br> [ImportMany(typeof(IVirtualHost))]<br> private IEnumerable<ivirtualhost> VirtualHosts;<br><br> internal VirtualHostContainer()<br> {<br> var path = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);<br> var dlls = new DirectoryInfo(path).GetFileSystemInfos("*.dll");<br> var catalog = new AggregateCatalog();<br> foreach (var dll in dlls.Where(d =&gt; !(d.Name.StartsWith("System") || d.Name.StartsWith("Microsoft"))))<br> {<br> try<br> {<br> var asm = Assembly.LoadFrom(dll.FullName);<br> var assemblyCatalog = new AssemblyCatalog(asm);<br> if (assemblyCatalog.Parts.Count() &gt; 0)<br> {<br> catalog.Catalogs.Add(assemblyCatalog);<br> }<br> }<br> catch (Exception)<br> {<br> }<br> }<br> new CompositionContainer(catalog).SatisfyImportsOnce(this);<br> }<br><br> internal List<string> GetHostTypes()<br> {<br> return (from h in VirtualHosts<br> select h.HostType).ToList();<br> }<br><br> internal IVirtualHost GetHost(string hostType, string hostName, string domain = "", string userName = "", string password = "")<br> {<br> var host = VirtualHosts.Single(h =&gt; h.HostType == hostType);<br> host.Connect(hostName, domain, userName, password);<br> return host;<br> }<br> }<br>}<br></string></ivirtualhost>

Above is the container code. The VirtualHosts property is attributed with an [ImportMany] attribute. The MEF framework will inject any classes that are attributed with the matching [Export] attribute into this property. This is done on line 28 in the call to SatisfyImportsOnce().

Once that property is populated, the GetHostTypes() and GetHost() methods are trivial. What’s a little trickier is dynamically loading the assemblies that may (or may not) contain IVirtualHost implementations. For that I just get the current assembly’s location and then try to load each dll (that doesn’t start with Microsoft or System). I create a new AssemblyCatalog from the assembly, and if there are any MEF exports (parts) in the assembly, I add the catalog to an AggregateCatalog. Once I’ve got all the AssemblyCatalogs into the AggregateCatalog, I create a CompositionContainer and tell it to make any hook ups using the SatisfyImportsOnce call on the VirtualHostContainer itself.

## Virtual Host Catalog

I’ve wrapped this class into a static class that you can call to get VirtualHosts. Here it is:

    namespace HybridLab.Virtual.Interfaces<br>{<br> public sealed class VirtualHostCatalog<br> {<br> public static List<string> GetHostTypes()<br> {<br> return new VirtualHostContainer().GetHostTypes();<br> }<br><br> public static IVirtualHost GetHost(string hostType, string hostName, string domain = "", string userName = "", string password = "")<br> {<br> return new VirtualHostContainer().GetHost(hostType, hostName, domain, userName, password);<br> }<br> }<br>}<br><br></string>

## Implementing IVirtualHost and IVirtualMachine

These interfaces are really simple (like all good interfaces). When I was implementing them for HyperV (on Windows 8) I realized that you can have a simple interface and a complicated implementation. I suppose that’s the power of interfaces – the interface consumer doesn’t have to care.

    namespace HybridLab.Virtual.Interfaces<br>{<br> public interface IVirtualHost<br> {<br> string HostName { get; }<br> string Domain { get; }<br> string UserName { get; }<br> string Password { get; }<br> string HostType { get; }<br> <br> void Connect(string hostName, string domain, string userName, string password);<br><br> List<ivirtualmachine> GetVMs();<br> }<br>}<br></ivirtualmachine>

The IVirtualHost has a couple of properties – the host type is used dynamically to discover host types. The rest are connection properties – the host name and admin credentials to the host machine. The Connect() method gets called in the VirtualHostContainer, so you don’t actually ever need to call it yourself. The final method is the method to get the list of VMs on the host.

I’ve also supplied an abstract HostBase class to get you started:

    namespace HybridLab.Virtual.Interfaces<br>{<br> public abstract class HostBase : IVirtualHost<br> {<br> public string HostName { get; protected set; }<br> public string Domain { get; protected set; }<br> public string UserName { get; protected set; }<br> public string Password { get; protected set; }<br><br> public virtual string HostType { get; protected set; }<br><br> public virtual void Connect(string hostName, string domain, string userName, string password)<br> {<br> HostName = hostName;<br> Domain = domain;<br> UserName = userName;<br> Password = password;<br> }<br><br> public virtual List<ivirtualmachine> GetVMs()<br> {<br> throw new NotImplementedException();<br> }<br> }<br>}<br></ivirtualmachine>

The IVirtualMachine interface is also fairly simple:

    namespace HybridLab.Virtual.Interfaces<br>{<br> public interface IVirtualMachine<br> {<br> void ApplySnapshot(string snapshotName);<br><br> void CreateSnapshot(string snapshotPrefix);<br><br> IVirtualHost Host { get; }<br><br> string Name { get; }<br><br> string DnsName { get; }<br><br> List<string> Snapshots { get; }<br> }<br>}<br></string>

There’s a reference to the VMs IVirtualHost, a name, a property that returns all the snapshots and the DnsName of the guest OS. Then there are ApplySnapshot() and CreateSnapshot() for doing those operations.

Once you’ve implemented an IVirtualHost and IVirtualMachine, simply add the MEF Export attribute onto your IVirtualHost implementation and drop the assembly into source control along with the interface and other assemblies for the Hybrid Lab Workflow. You won’t need to change the workflow at all.

Here’s an example TestHost that I implemented for testing the VirtualHostCatalog class:

    namespace HybridLab.Virtual.TestHost<br>{<br> [Export(typeof(IVirtualHost))]<br> public class TestHost : HostBase<br> {<br> public override string HostType<br> {<br> get<br> {<br> return "Test";<br> }<br> }<br> }<br>}<br>

Note the [Export] attribute decorating the class. Of course you’ll need some more code for a real host implementation!

That’s all there is to it. If you’re going to be implementing a host and need some help, let me know! Also, I can add it to the [Hybrid Lab Workflow project on Codeplex](http://hybridlabworkflow.codeplex.com/).

Happy implementing!

