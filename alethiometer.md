# Computation as bit routing
No objects, no classes, type system defines behavior. Behaviors that bind into coherent (hoepfully) action. Remote/local is mostly load balance.

Defining a problem and writing a program are the same thing, how behavior routes. A convoluted routing scheme is a poorly understood problem. Query/lookup isn't locating its behavior resolution. A hard to find object is a hard to describe object.

/home/{domain}/.policy is where rules are defined, think rpm dependency for arbitrary behavior. Most things are rules. /home/{domain}/.bind is basically projection. So say im working on proj x with team y. the 'x' is a domain /home/x where all of it's information lives. But I don't work in x. I work in /home/me/.bind/x. So I have my own projection of x, can reorganize, create alternative auth and hosting structures, whatever is negotiated between /home/x and /home/me/.bind/x as defined in both .policy systems. It also means /home/x/.bind is a sort of import/export plane such that /home/x/.policy can reference /home/me components as /home/x/.bind/me. Probably best to thinkg of .policy as firewall/rule table and .bind is the request+response.

"Space" is the dimensionality of a problem domain. Rasters convert scene geometry to pixels, whether the dimensions are cartesian (cad), dom, product data.

- .policy -> routing rules
- .bind -> materialized state,
   - binding point of composable objects

Policies are something like config files. If you define hand edited or static configurations, they look like data nodes. But if they are contingent rules, they look more like transforms/functions.

- .policy
    - gateway|gw -> address of a contactable node 
        - how to reach behavior cluster
    - acs -> authentication, identity, roles
    - domain -> range, membership, defines scope
    - rms -> address of an inert node
        - inventoriable

- gw.wayland|x11 -> wayland|x11 server address
- gw.doc -> the address of a doc rend agent
    - html
    - js
    - pdf
    - latex
    - docx

- space -> the geometry of the problem sapce
    - tk -> widgets
    - latex -> 
    - html -> dom
    - ...

## Bus

All action is communication across a bus. Even a disk, is just potential sitting on a bus not connected to anything that would drain it. So a gateway is just a named bus. Buses can be complex, holding potential value that resolves once connected to an endpoint. Like an internet connection can be a standin for whatever ip atcually resolves, the logic depending on it doesn't care whats underneith it. This gives monadic composition and lazy evaluation as routing potential.

## Dependency

- requires
- permits
- excludes
- provides
- supliments
- replaces
- prefers -> bidirectional

## Network

- 10.0.0.1\16:provides:ucryo.lan -> dhcp range
- ucryo.lan:requires:(10.0.0.1,10.0.0.2)
- scrooge:provides:10.0.0.1 -> dns entry
- dewey:provides:10.0.0.2 -> dns entry
- scrooge:requires:acs.signature.ucryo.lan[0]
- dewey:requires:acs.signature.ucryo.lan[1]
- mac1:provides:acs.signature.ucryo.lan[0]
- mac2:provides:acs.signature.ucryo.lan[0]

## Graphics

- "Table name":provides:dom.el1.name
- dom.el1:provides:table
- table:provides:cols
- table:provides:rows

- edit.src -> vi editor
    - database/graph editor
- edit.dst -> wysiwyg
    -rendered output

