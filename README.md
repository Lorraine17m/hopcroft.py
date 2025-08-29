# hopcroft.p
from collections import defaultdict
from typing import Set, Dict, Tuple
from dfa import DeterministicFiniteAutomaton

def hopcroft_minimize(dfa: DeterministicFiniteAutomaton) -> DeterministicFiniteAutomaton:
    states = set(dfa.states)
    accept = set(dfa.accept_states)
    non_accept = states - accept
    P = [accept, non_accept]
    W = [accept.copy()]

    inverse_transitions = defaultdict(lambda: defaultdict(set))
    for (state, symbol), next_state in dfa.transitions.items():
        inverse_transitions[symbol][next_state].add(state)

    while W:
        A = W.pop()
        for symbol in dfa.alphabet:
            pre_image = set()
            for q in A:
                pre_image |= inverse_transitions[symbol][q]
            new_P = []
            for Y in P:
                intersect = Y & pre_image
                difference = Y - pre_image
                if intersect and difference:
                    new_P.extend([intersect, difference])
                    if Y in W:
                        W.remove(Y)
                        W.extend([intersect, difference])
                    else:
                        W.append(intersect if len(intersect) <= len(difference) else difference)
                else:
                    new_P.append(Y)
            P = new_P

    state_map = {}
    new_states = []
    new_accept = set()
    new_transitions = {}
    new_start = None

    for i, group in enumerate(P):
        name = f"S{i}"
        new_states.append(name)
        for state in group:
            state_map[state] = name
        if dfa.start_state in group:
            new_start = name
        if group & accept:
            new_accept.add(name)

    for old_state in states:
        new_state = state_map[old_state]
        for symbol in dfa.alphabet:
            target = dfa.transitions[(old_state, symbol)]
            new_target = state_map[target]
            new_transitions[(new_state, symbol)] = new_target

    return DeterministicFiniteAutomaton(
        states=new_states,
        alphabet=dfa.alphabet,
        start_state=new_start,
        accept_states=new_accept,
        transitions=new_transitions
    )

