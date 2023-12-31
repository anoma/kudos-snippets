---
hide:
  - navigation
---

# Kudos as resources in Juvix (WIP)

This code was typechecked using Juvix v0.5.4.

```juvix 
module kudos;

import Stdlib.Prelude open;
import Data.Set as Set open; 
```

## Basic types

```juvix
syntax alias Predicate := Nat;
syntax alias Hash := Nat;
syntax alias Field := Nat;

syntax alias Signature := Nat;
syntax alias Key := Nat;

Byte : Type := List Nat;

Bytes : Type := List Byte;
```


## Custom types

```juvix
type Owner :=
  mkOwner {
    identity : ExternalIdentity;
    signature : Signature
  };

type ExternalIdentity := mkExternalIdentity {key : Key};
```

```juvix
module Kudo;
  
  type Resource :=
    mkResource {
      label : Hash;
      predicate : Predicate;
      quantity : Field;
      value : Value
    };

  type Value :=
    mkValue {
      owner : Owner;
      originator : Signature
    };

  Env : Type := Set Predicate;
end;
```

```juvix
trait
type isVerifiable (A : Type) :=
  mkVerifiable {verify : A -> Bool};

open isVerifiable;

instance
valueIsVerifiable : isVerifiable Kudo.Value :=
  mkVerifiable (verify := \ {_ := true});
```

```juvix
kudoIsVerifiable : isVerifiable Kudo.Resource :=
  mkVerifiable
    (verify := \ {k := verify (Kudo.Resource.value k)});
```

### Partial transactions


```juvix
type PartialTransaction :=
  mkPartialTransaction {
    input : List Kudo.Resource;
    output : List Kudo.Resource;
    extra : Bytes
  };
open PartialTransaction;

syntax alias PTX := PartialTransaction;
```

## Evaluation context

```juvix
axiom env : Kudo.Env;
axiom ! : {A : Type} -> A;

kudo_logic (ptx : PartialTransaction) : Bool :=
  let
    checkInputs : Bool :=
      all
        λ {ri :=
          let
            cond1 : Bool := member? (Kudo.Resource.predicate ri) env;
            cond2 : _ := true;
            cond3 : _ := true;
          in cond1 && not (cond2 || cond3)}
        (input ptx);

    checkOutput : Bool :=
      all
        λ {ri :=
          let
            cond1 : Bool := member? (Kudo.Resource.predicate ri) env;
            cond2 : _ := true;
            cond3 : _ := true;
          in cond1 && not (cond2 || cond3)}
        (output ptx);
    checkConnectiveValidity : Bool := true;
  in checkInputs && checkOutput && checkConnectiveValidity;

-- insert kudo_logic in the set, 
-- or better use a map instead

instance
partialVerifiable : isVerifiable PartialTransaction :=
  mkVerifiable (verify := kudo_logic);
```

```juvix
axiom agentA : Owner;
axiom sig_agentA1 : Signature;
axiom type_sig_agentA1 : Signature;
axiom sig_agentA2 : Signature;
axiom type_sig_agentA2 : Signature;
axiom agentB : Owner;
axiom sig_agentB1 : Signature;
axiom type_sig_agentB1 : Signature;
axiom hash : {A : Type} -> A -> Hash;

valueA1 : _ :=
  Kudo.mkValue (owner := agentA; originator := sig_agentA1);

kudoA1 : _ :=
  Kudo.mkResource
    (predicate := !;
    -- kudo_logic,
    label := hash agentA;
    quantity := 1;
    value := valueA1);

valueA2 : _ :=
  Kudo.mkValue (owner := agentA; originator := sig_agentA2);

kudoA2 : _ :=
  Kudo.mkResource
    (predicate := !;
    -- kudo_logic,
    label := hash agentA;
    quantity := 1;
    value := valueA1);

valueB1 : _ :=
  Kudo.mkValue
    (owner := agentB; originator := type_sig_agentB1);

kudoB1 : _ :=
  Kudo.mkResource
    (predicate := !;
    -- kudo_logic,
    label := hash agentB;
    quantity := 1;
    value := valueB1);

axiom Float : Type;

syntax alias Weight := Float;

type ResourceIntentElement :=
  | Resource
  | WantResourceType;

axiom Term : Type;

syntax alias Connective := Term;
```

### Intents over Kudos

```juvix
type ResourceIntent :=
  mkIntent {
    have : List (Weight × ResourceIntentElement);
    want : List (Weight × ResourceIntentElement);
    connectives : Connective
  };

type KudoIntentElement :=
  | KudoResource
  | WantKudoResourceType;

{-- Each Agent can produce intents over which Kudos they have and which Kudos
  they want, for arbitrary size vectors of both. --}
type KudoIntent :=
  mkKudoIntent {
    have : List (Weight × KudoIntentElement);
    want : List (Weight × KudoIntentElement);
    connectives : Connective
  };
```

