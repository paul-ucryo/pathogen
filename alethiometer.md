Computation as bit routing. No objects, no classes in the type syste, just behaviors that bind into coherent (hoepfully) action. Remote/local is mostly load balance.

Defining a problem and writing a program are the same thing, how behavior routes. A convoluted routing scheme is a poorly understood problem. Query/lookup isn't locating its behavior resolution. A hard to find object is a hard to describe object.

.policy is where most stuff is defined and exists. .bind is basically projection. So say im working on proj x with team y. the 'x' is a domain /home/x where all of it's information lives. i don't work in x. i work in /home/me/.bind/x. So i have my own projection of x, can reorganize, create alternative auth and hosting structures, whatever is negotiated between /home/x and /home/me/.bind/x. It also means /home/x/.bind is a sort of import/export plane such that /home/x/.policy can reference /home/me components as /home/x/.bind/me.

Space is the dimensionality of a problem space. rasters convert scene geometry to pixels, whether the dimensions are cartesian (cad), dom, product data.

.policy -> routing rules
.bind -> materialized state,
    ->binding point of composable objects

Policies are something like config files. If you define hand edited or static configurations, they look like data nodes. But if they are contingent rules, they look more like transforms/functions.

.policy
    gateway|gw -> address of a contactable node 
        ->how to reach behavior cluster
    acs -> authentication, identity, roles
    domain -> range, membership, defines scope
    rms -> address of an inert node
        ->inventoriable

gw.wayland|x11 -> wayland|x11 server address
gw.doc -> the address of a doc rend agent
    html
    js
    pdf
    latex
    docx

space -> the geometry of the problem sapce
    tk -> widgets
    latex -> 
    html -> dom
    ...
