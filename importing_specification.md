
# Summary

We propose adding an importing mechanism to the WGSL shading language as an extension.

# Motivation

When writing bigger WGSL shaders, one tends to **split the code into reusable pieces**. Examples are re-using a set of lighting calculation functions and sharing a struct between two compute shaders and.

However, current WGSL tooling is not built for this. The official language purposefully does not include this feature, and there is no standardized extension yet. 

One important aspect is getting buy-in from **WGSL language servers**, as shader development with IDEs should be well supported.

We also should account for **importing shader from libraries**. Ideally, users could upload WGSL shaders to existing package managers, which other users could then consume. 

Finally, we want **multiple tools** which can compile WGSL-with-imports down to raw WGSL. We should not make it ecosystem specific.

TODO: How should the importing syntax work?
- We need a language server to be able to discover shaders *without* scanning all files. 
  - Either it can look at an import, and figure out the location. (~relative imports)
  - Or there's another file that tells it how to resolve imports. (~import maps and Cargo.toml)
- Portable. We shouldn't pick a syntax that is platform-specific
  - For instance, relying on symbols that aren't valid in Windows file names is not ideal.
- Not dump everything in the global namespace.
- Have a syntax for importing a function with an already used name.
  - Import renaming, or fully qualified paths would work. But remember that I don't ever want to have to write `module::submodule::anotherone::help::pls::my_func()`.
- Granular. It should be possible to only import certain things.
  - Either scoping like `import module` followed by `module::my_func()` or granular imports `import my_func from module` would do the job.
- Have re-exports or similar. I never want to be forced to write `module::submodule::anotherone::help::pls::my_func()`.
- Not conflict with WGSL syntax

TODO: Look up at how other programming languages justify their features.
TODO: Ask the naga_oil peeps.

# Guide-level explanation

Explain the proposal as if it was already included in the language and you were teaching it to another Rust programmer. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- Explaining how Rust programmers should think about the feature, and how it should impact the way they use Rust. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.
- If applicable, describe the differences between teaching this to existing Rust programmers and new Rust programmers.
- Discuss how this impacts the ability to read, understand, and maintain Rust code. Code is read and modified far more often than written; will the proposed feature make code easier to maintain?

For implementation-oriented RFCs (e.g. for compiler internals), this section should focus on how compiler contributors should think about the change, and give examples of its concrete impact. For policy RFCs, this section should provide an example-driven introduction to the policy, and explain its impact in concrete terms.

# Reference-level explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Drawbacks

Why should we not do this?

# Rationale and alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?
- If this is a language proposal, could this be done in a library or macro instead? Does the proposed change make Rust code easier or harder to read, understand, and maintain?

## Not agreeing on a standard

One major upside of standardizing it is that it becomes practical for language servers to support it. 

The usual alternative is that one library, like shaderc, becomes very popular and the standard ends up being "whatever popular library XYZ does".

An open process lets us find a better solution.

## Preprocessor `#include <lighting.wgsl>` 

One alternative, which is common in the GLSL and C worlds, is an including mechanism which simply copy-pastes existing code. A major upside is that this is very simple to implement.

One drawback is that importing the same shader multiple times, which can also happen indirectly, does not work without other preprocessor features. 

```c
// A.wgsl
#include <lighting.wgsl>
#include <math.wgsl>
```
```c
// lighting.wgsl
#include <math.wgsl>
```
would not work, since anything defined in `math.wgsl` would be imported twice. In C-land, this is solved by using include guards. (TODO: check if the name is correct)


Another drawback is that using the same name twice is impossible. In C-land, this leads to pseudo-namespaces, where major libraries will prefix all of their functions with a few symbols. An example of this is the Vulkan API `vkBlabla` (TODO: examples from vk API)

A future drawback is that "privacy" or "visibility" becomes very difficult to implement. Everything that is importing is automatically public and easily accessible.
In C-land, the workaround is header files. In other languages, such as Python, the convention ends up being "anything prefixed with an underscore `_` is private".

## Using a higher level language

There are multiple higher level shading languages, such as slang (TODO: link) or Rust-GPU (TODO: link) which support imports. They also support more features that WGSL currently does not offer. For complex projects, this can very much pay off.

The downside is using additional tooling, and dealing with an additional translation layer. An additonal translation layer could lock shader authors out of certain WGSL features.

This is always a trade-off, and I believe that offering multiple solid choices is the ideal option.

# Prior art

Discuss prior art, both the good and the bad, in relation to this proposal. A few examples of what this can include are:

- For language, library, cargo, tools, and compiler proposals: Does this feature exist in other programming languages and what experience have their community had?
- For community proposals: Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the lessons from other languages, provide readers of your RFC with a fuller picture. If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other languages.

Note that while precedent set by other languages is some motivation, it does not on its own motivate an RFC. Please also take into consideration that rust sometimes intentionally diverges from common language features.

# Unresolved questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

# Future possibilities

- Doc comment syntax
- pub, private, ...
- integrate with source maps (WGSL has those, right?)

Think about what the natural extension and evolution of your proposal would be and how it would affect the language and project as a whole in a holistic way. Try to use this section as a tool to more fully consider all possible interactions with the project and language in your proposal. Also consider how this all fits into the roadmap for the project and of the relevant sub-team.

This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related.

If you have tried and cannot think of any future possibilities, you may simply state that you cannot think of anything.

Note that having something written down in the future-possibilities section is not a reason to accept the current or a future RFC; such notes should be in the section on motivation or rationale in this or subsequent RFCs. The section merely provides additional information.