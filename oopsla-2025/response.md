

Thank you for your careful readings and thoughtful comments! We are heartened by the encouraging comments and strong overall assessment from the reviewers. Below we address the questions and concerns of RA and RB. Some comments just point out small errors - we do not address these below, but will fix the errors for the final submission. If we do not address a comment, it is because we agree with it and will fix the issue. 

# Review A

> I'm excited to see if I can apply some of these ideas in my own work proof assistants [...]

We too are specifically interested in an incremental dependent type checker. We would love to converse about this should we meet at the conference!

> My main criticism of the paper is that it is just too hard to read the formalism [...]

We settled on this notation after a few iterations. The left-to-right order of information flow clarifies the update propagation steps, putting analyzed types on the left of an expression and synthesized types on the right. The lavender for type annotations allows the dark symbols of the actual program to stand out. Any less compact notation would cause the small examples to take multiple lines, making them harder to read, and any more compact a notation would make it hard to read the annotations. 

We will replace the mapsto symbol in function abstractions to reduce the number of arrows. 

> Line 138: did you mean \rhd? (I'm comparing with Figure 1.)

We meant \lhd, the error was in Figure 1. It is a reversed pipeline operator, with the function preceeding the argument. 

> Figure 1: A marked analytic expression has an optional type and then a marked synthetic expression—which has another optional type. Why so many types? Is the point that when there is a type error, the two types could be different and we need to remember what they are?

There are two optional types on each node of the program syntax tree, one for the analyzed/expected type and one for the synthesized/inferred type. We need to remember what they both are because they propagate differently through the program (analyzed types move down into subterms, synthesized types move up), and, as you say, if they are different then we want the user to have access to both. 

> Line 369: [...] I couldn't even find in the formal section later a specification of the NewAna rule.

We called the rule "NewAna" in the example section, but "StepAna" later. We will fix this. 

> Figure 6. The negation in premise of the final update propagation rule might be a little dodgy, if one is trying to give an inductive definition. It this shorthand for an additional relation that expresses that a given program is terminal?

This is not shorthand for a different relation, and the Agda mechanization formalizes the premise as a literal negated existential. The idea is to directly express that updates have propagated until they can't anymore. I don't think this poses a problem for the inductively defined relation; the relation is defined as an inductive type, not defined by induction on programs.

> Line 690: I am having trouble seeing how the premise of the InsideStep rule is different from the conclusion. [...]

We were using the long mapsto arrow to denote update propagation on programs, and the regular mapsto arrow to denote update propagation on (analytic and synthetic) expressions. Since programs are defined as a subset of analytic expressions, the terms in the premise and conclusion of the rule are the same (or if you prefer, look the same but are considered different sorts).

We agree that this is confusing. We will fix this by choosing more distinct notation for the update propagation relation on programs. This must remain a distinct relation because it includes extra transitions (TopStep).

> I noted that the malcom-workbench directory contained only the build artifacts and not the actual source code. Was this intentional?

The source code is in the `malcom-workbench/default` directory. 

# Review B

> Line 82: That's a big number! I'm not sure it's very realistic, though. Reading through the evaluation section at the end, this is what is achieved if type-checking is done after every edit. With a whole-program type checker, doing that is infeasible. And thus there is no process that gets 275.96 times faster with this work. Instead, the result (I believe) is something more like that typing information will be available to the user x% more of the time, where that could be computed by seeing how many of the type-checking passes are newly under 200ms (or some threshold). 

This is a good point: whether this number accurately reflects actual speedup depends on our assumptions about the architecture of the incremental and non-incremental systems. Our comparison was with an editor in which the core loop was something like: recieve edit, type check, display, repeat. Your suggested metric assumes something more like an independently edited program buffer, with a background type checking task dispatched on each edit. The latter may well be more like programming environments we are used to. With the former, whole-program type checking is indeed infeasible, but that infeasibility manifests to the user, for example, as a 276x editing slowdown!

> Line 99: The paper contrasts bidirectional type-checking with unification based. But the type-checkers I know do both. Critically, MALC has only annotated lambdas. Does your approach work with unannotated lambdas? 

MALC is designed so that unannotated lambdas can be implemented as syntactic sugar for lambdas annotated with the unknown type. The analytic typing rule for lambdas allows the codomain of the analyzed type to continue into the body of the lambda, simulating one aspect of the analytic rule for unannotated lambdas. The other aspect, in which the type of the bound variable is determined by the expected domain type, is absent from our system for simplicity, but could be added without trouble. 

> Line 138: I think the triangle should point the other way.

See RA above.

> Rule MarkAnaFun: I found this rule surprising -- or, rather, that there is always a checkmark toward the left of the conclusion. I think maybe there's an invariant that the mark to the left of an abstraction is always a checkmark, because the relevant mark is really in the abstraction itself? 

You're right. The mark to the left of the abstraction represents the inconsistency between the analyzed and synthesized type, and since abstractions are subsumable (they have a special rule for being analyzed against a type), they never synthesize a type while being analyzed against a type, and this mark is always a checkmark. The language is defined this way for uniformity. That mark to the left of the abstraction is not part of the abstraction form, but the analytic expression form, which contains an arbitrary constructor (many of which are subsumable, and which need this mark). It is a valid alternative to give every subsumable form a consistency mark instead, but this causes a lot of logic to be duplicated. 

> Definition 3.1. I found this unsatisfying. The definition just demarcates the codomain of the type-checker. Why is this useful? I was expecting a set of rules that assert some consistency among the various annotations and marks.

It does more than demarcate the codomain of the type checker; it says that a marked program is well-marked if erasing type information form it, then running the type checker on it, results in the same original marked program. Well-marked programs are those for which erasure is a right inverse of marking. 

> Definition 3.1. Is there a missing ∅⊢ in the conclusion?

No, this is using the marking judgment for programs (see the last box in Figure 2).

> Equation (5): Why are the annotations around x dirty?

Because when an expression is wrapped in a function application, 1) it is now being analyed against no type, which it might not have been before, so that type must be dirtied, and 2) the type it synthesizes must be propagated along to determine the expected type of the argument and the synthesized type of the application, so must be dirtied. 

> Fig. 6: In the rightmost rule, there needs to be a condition that p' != p. 

A program never steps to itself, so this is not necessary.

> Page 15: I got a little confused in here. Does the length of the rightward arrow matter? Rule InsideStep seems awfully boring if it doesn't, but I think I missed where this was explained.

See the answer to review A concerning "Line 690." TL;DR: yes, the length of the arrow does matter, but we will switch to less-confusing notation. 

> Page 15: By this point, I got a little bored. The paper patiently explains each rule. That's helpful for an implementor. But as a paper-reader, it's not what I want at this point. Instead, I'd want some high-level understanding of how to take my favorite type system and incrementalize it. That is, what general techniques did you figure out were the right ones for transforming a traditional type system into an incremental one? That stuff is really interesting! Much more so than the details of the rules (which might go in an appendix).

Noted. We will expand the section on generalizing our approach and signpost that some readers may want to skip some of the explanations of particular rules and refer to that section. 

> One question I feel wasn't quite answered: is the implementation interactive? That is, can I just type away and see all of this in action? Or do I need to feed the implementation with a string of edit descriptions and have it play them back at me? 

Somewhere in between. You can interactively press HTML buttons that trigger actions, which is admittedly less fun that being able to type. In ongoing work we are integrating this approach into Hazel, which has a more usable editor. 

> Page 24: Do you plan on submitting the artifact for evaluation? If not, why not?

We do. 

> Do you see any obstacles in incrementalizing a type system that supported un-annotated lambdas?

See the response above to the comment concerning "Line 99." 

> Do you have high-level observations about incrementalizing that could be included in revisions?

We do, and we will expand on this. For instance, we have since extended the approach to include System F polymorphism, although type-level computations are still not incrementalized. 