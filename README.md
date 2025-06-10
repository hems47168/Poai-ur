# Poai-ur
class Term:
    def __init__(self, name, args=None):
        self.name = name
        self.args = args if args else []

  def __str__(self):
        if self.args:
            return f"{self.name}({', '.join(map(str, self.args))})"
        else:
            return self.name

class Clause:
    def __init__(self, literals):
        self.literals = literals

  def __str__(self):
        return f" ∨ ".join(map(str, self.literals))

class Literal:
    def __init__(self, predicate, args=None, negated=False):
        self.predicate = predicate
        self.args = args if args else []
        self.negated = negated

  def __str__(self):
        if self.negated:
            return f"¬{self.predicate}({', '.join(map(str, self.args))})"
        else:
            return f"{self.predicate}({', '.join(map(str, self.args))})"

def unify(term1, term2, substitution=None):
    if substitution is None:
        substitution = {}
    if term1.name == term2.name and len(term1.args) == len(term2.args):
        for arg1, arg2 in zip(term1.args, term2.args):
            if not unify(arg1, arg2, substitution):
                return False
        return True
    elif term1.name.islower() and term2.name.isupper():
        if term1.name in substitution:
            return substitution[term1.name] == term2
        else:
            substitution[term1.name] = term2
            return True
    elif term1.name.isupper() and term2.name.islower():
        if term2.name in substitution:
            return substitution[term2.name] == term1
        else:
            substitution[term2.name] = term1
            return True
    else:
        return False

def resolve(clause1, clause2):
    for literal1 in clause1.literals:
        for literal2 in clause2.literals:
            if literal1.predicate == literal2.predicate and literal1.negated != literal2.negated:
                substitution = {}
                if unify(Term(literal1.predicate, literal1.args), Term(literal2.predicate, literal2.args), substitution):
                    new_clause = Clause([literal for literal in clause1.literals + clause2.literals if literal != literal1 and literal != literal2])
                    new_clause.literals = [apply_substitution(literal, substitution) for literal in new_clause.literals]
                    return new_clause
    return None

def apply_substitution(literal, substitution):
    new_args = [substitute_term(arg, substitution) for arg in literal.args]
    return Literal(literal.predicate, new_args, literal.negated)

def substitute_term(term, substitution):
    if term.name in substitution:
        return substitution[term.name]
    else:
        new_args = [substitute_term(arg, substitution) for arg in term.args]
        return Term(term.name, new_args)

# Example usage
clause1 = Clause([Literal("P", [Term("x")]), Literal("Q", [Term("y")], negated=True)])
clause2 = Clause([Literal("P", [Term("a")], negated=True), Literal("R", [Term("z")])])

new_clause = resolve(clause1, clause2)
if new_clause:
    print("Resolved clause:", new_clause)
else:
    print("No resolution possible")
