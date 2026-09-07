# Welcome to Retouched Documentation

This documentation describes the Brass Monkey protocol (1.7.0) which was reverse-engineered from the original applications.  
Brass Monkey was a service that allowed games to be played on a TV or monitor using your phone as a controller.  

Brass Monkey was developed by Infrared5 Inc. in circa 2011 and was discontinued in 2015.   
Nitrome Touchy was a fork/reskin of Brass Monkey made by Nitrome Ltd. in partnership with Infrared5 Inc. It was released in 2012 and was discontinued when Brass Monkey was.  

Retouched is an open-source project dedicated to reviving this functionality, it provides several implementations:

- **Bronze Monkey**: A transport-agnostic Rust library that re-implements the Brass Monkey SDK.
- **Retouched Server**: A Rust-based re-implementation of the Brass Monkey Server.
- **Retouched Flutter**: A Flutter-based re-implementation of the controller app.
- **Retouched Web**: A React-based web re-implementation of the controller app.

This documentation is for anyone who wants to understand the protocol, contribute to Retouched, or create their own implementations.

The documentation is structured as follows:

- Protocol Specification: A detailed breakdown of the protocol.
- Connection Flows: A description of the connection lifecycle.
- Reference Tables: Useful tables for quick reference.

This documentation assumes the reader has a basic understanding of networking and programming concepts.

!!! note
    **This documentation is a work in progress!**

## License

The documentation is licensed under the Creative Commons Attribution 4.0 International License.  
See the [LICENSE-CONTENT.md](https://github.com/Retouched-Project/retouched-project.github.io/blob/main/LICENSE-CONTENT.md) file for details.