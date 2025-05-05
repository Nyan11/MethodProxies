# MethodProxies

A library to decorate and control method execution. Such library allows one to proxy **any** method in the system and its architecture is robust enough to develop powerful propagating method proxies that automatically install them without falling in endless loops.

## How to load

```st
EpMonitor disableDuring: [
	Metacello new
		baseline: 'MethodProxies';
		repository: 'github://Nyan11/MethodProxies/src';
		load. ]
```

#### How to depend on this project

```st
spec 
    baseline: 'MethodProxies' 
    with: [ spec repository: 'github://pharo-contributions/MethodProxies/src' ].
```

## Examples

A simple counting handler.

```st
handler :=  MpCountingHandler new.
p := MpMethodProxy 
	onMethod: Object >> #error: 
	handler: handler.
p install.
p enableInstrumentation.
1 error: 'foo'.
p uninstall.
handler count.
>>> 1
```

A simple example showing that an handler may failed still the system does not destroy Pharo.

```st
"Managing exceptions in the spyied method"

p := MpMethodProxy 
        onMethod: MwClassB >> #methodTwo 
	handler: MpFailingHandlerMock new.
p install.
p enableInstrumentation.
p uninstall.
MpClassB new methodTwo.
```

### Sharing Handlers
The design of method proxies supports the sharing of handlers between multiple proxied methods. 
For example the following example shows how we can monitor and gather information about `new` and `new:` in the same place

```st
h := MpAllocationProfilerHandler new.
p1 := MpMethodProxy 
	onMethod: Behavior >> #basicNew 
	handler: h.
p1 install.
p2 := MpMethodProxy 
	onMethod: Behavior >> #basicNew: 
	handler: h.
p2 install.
p3 := MpMethodProxy 
	onMethod: Array class >> #new: 
	handler: h.
p3 install.

p1 enableInstrumentation.
p2 enableInstrumentation.
p3 enableInstrumentation.

p1 uninstall.
p2 uninstall.
p3 uninstall.
```

### Propagation proxies

A more advanced and challenging for the architecture of method proxies is a propagating handler. 
The idea is simple, before a method executes, all the implementor methods of its messages are proxified with propagating handlers.
So instead of proxifying the complete system only we subpart is spyied. This is this challenging because the infrastructure should make sure that we are not proxifying the code that is proxifying the system else we would end up in a severe and endless loop. 
The handler design protects you from that by avoiding that you extend and break the clear separation between base and meta-level.

```st
testCase := StringTest selector: #testAsCamelCase.
(MpMethodProxy 
   onMethod: testCase testMethod 
   handler: MpProfilingHandler new) install; enableInstrumentation.
testCase run.

proxies := MpProfilingHandler allInstances.
proxies do: #uninstall.
```

## Some Archeology and History

Method Wrappers were originally developed by John Brant for proprietary software. You can read "Evaluating Message Passing Control" article from S. Ducasse to understand the original implementation and a comparison with other approaches. What you see is that back in 1998/9 there were already multiple ways to control message passing. Method Wrappers is one of them. 

MethodWrappers were basically using CompiledMethod prototypes that were patched (cloned + adding selector/class) during their application because the VM of the system where Method Wrappers were implemented did not support to have anything but CompiledMethod as value of method dictionaries. 
This is not the case for Pharo. And the design and implementation of MethodProxies allows us to wrap any part of the system and in addition to design propagating proxies without blowing up the system. This is a key properties for us.

**Note for grumpies.** For the people that could think that we would like to steal this idea, we encourage them to lower their paranoia by having a look at our CV and publication record list. We redeveloped from scratch this library while taking into account the original intention and we wrote many tests to validate them. Now we may have missed some points. If this is the case please be kind and report it to us. 

## Bibliography

- Jordan Montaño S., Sandoval Alcócer J., Polito G., Ducasse S., Tesone P., MethodProxies: A Safe and Fast Message-Passing Control Library, IWST '24 June 2024, France. [PDF](https://hal.science/hal-04708729v1/document)
- Stéphane Ducasse, Evaluating Message Passing Control Techniques in Smalltalk, Journal of Object-Oriented Programming (JOOP), 12, 39–44, SIGS Press, 1999, Impact factor 0.306. [PDF](http://rmod-files.lille.inria.fr/Team/Texts/Papers/Duca99aMsgPassingControl.pdf)
- John Brant, Brian Foote, Ralph E. Johnson, and Donald Roberts. Wrappers to the Rescue. In Proceedings of European Conference on Object-Oriented Programming (ECOOP). Springer, Berlin, Heidelberg, 1998. http://www.laputan.org/brant/brant.html (https://link.springer.com/chapter/10.1007/BFb0054101)


