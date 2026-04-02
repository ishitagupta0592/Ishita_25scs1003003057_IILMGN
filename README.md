# Ishita_25scs1003003057_IILMGN
Gene editing in wildlife offers solutions for conservation but raises major challenges. Ecological risks, ethical concerns, lack of proper laws, and chances of misuse make regulation difficult. Strong policies, scientific research, and global cooperation are needed to ensure safe and responsible use of this technology.
#!/usr/bin/env python3
"""
rbra.py
A simple interactive implementation of the 7-step Risk-Based Regulatory Algorithm (RBRA)
from the user's attached project PPT (Ishita Gupta). It evaluates readiness, highlights gaps,
and generates simple recommendations and a monitoring/policy template.

Usage:
    python rbra.py
    python rbra.py --example
"""

import argparse
import json
from textwrap import indent

# --- Helper scoring functions ---


def bool_score(flag: bool, weight: float = 1.0):
    """Return a 0-100 score based on boolean (0 or 100) scaled by weight."""
    return (100.0 if flag else 0.0) * weight


def scale_score(value: float, max_val: float = 10.0, weight: float = 1.0):
    """Map 0..max_val to 0..100 then apply weight (weight between 0 and 1)."""
    raw = max(0.0, min(value, max_val)) / max_val * 100.0
    return raw * weight


# --- Step functions following the PPT steps ---


def step1_scope_and_objectives(data):
    """
    Step 1: DEFINE scope & IDENTIFY objectives
    Checks whether alternatives assessed and necessity proven.
    """
    alt_assessed = data.get("alternatives_assessed", False)
    necessity_evidence = data.get("necessity_evidence", False)
    notes = []
    score = 0.0

    if alt_assessed:
        score += 60.0
    else:
        notes.append("No evidence that non-GE alternatives were fully assessed.")

    if necessity_evidence:
        score += 40.0
    else:
        notes.append("No clear necessity evidence (e.g., failure of existing methods).")

    return {"score": score, "notes": notes}


def step2_collect_scientific_data(data):
    """
    Step 2: COLLECT scientific_data, regulations, ethical_opinions
    Check completeness of scientific dossier, regulatory scan, and stakeholder inputs.
    """
    scientific_complete = data.get("scientific_data_complete", False)
    regulatory_scan = data.get("regulatory_scan", False)
    stakeholder_engagement = data.get("stakeholder_engagement", False)
    notes = []
    score = 0.0

    score += bool_score(scientific_complete, weight=0.5)
    score += bool_score(regulatory_scan, weight=0.3)
    score += bool_score(stakeholder_engagement, weight=0.2)

    if not scientific_complete:
        notes.append("Incomplete scientific data (e.g., proof of mechanism, lab/containment results).")
    if not regulatory_scan:
        notes.append("No comparison with existing national/international rules.")
    if not stakeholder_engagement:
        notes.append("Stakeholder consultation missing or insufficient.")

    return {"score": score, "notes": notes}


def step3_analyze_risks(data):
    """
    Step 3: ANALYZE ecological_risks, ethical_concerns, socio_economic_impacts
    Input expected: ecological_risk (0-10), ethical_risk (0-10), socioecon_risk (0-10)
    """
    eco = float(data.get("ecological_risk", 5.0))
    eth = float(data.get("ethical_risk", 5.0))
    soc = float(data.get("socioeconomic_risk", 5.0))
    notes = []

    eco_score = 1.0 - (eco / 10.0)  # more risk -> lower score
    eth_score = 1.0 - (eth / 10.0)
    soc_score = 1.0 - (soc / 10.0)

    # Weighted
    score = (eco_score * 0.5 + eth_score * 0.3 + soc_score * 0.2) * 100.0

    if eco >= 7.0:
        notes.append("High ecological risk flagged.")
    if eth >= 7.0:
        notes.append("High ethical concerns flagged.")
    if soc >= 7.0:
        notes.append("High socioeconomic impact flagged.")

    return {"score": score, "notes": notes}


def step4_mitigation_and_kill_switch(data):
    """
    Step 4: COMPARE regulations WITH risks & FIND gaps / Mitigation plan existence
    """
    mitigation_plan = data.get("mitigation_plan", False)
    kill_switch = data.get("kill_switch", False)
    response_capacity = data.get("response_capacity", False)
    notes = []
    score = 0.0

    score += bool_score(mitigation_plan, weight=0.5)
    score += bool_score(kill_switch, weight=0.3)
    score += bool_score(response_capacity, weight=0.2)

    if not mitigation_plan:
        notes.append("No detailed mitigation plan present.")
    if not kill_switch:
        notes.append("No 'kill-switch' or reversible control measures defined.")
    if not response_capacity:
        notes.append("No clear emergency response capacity (labs, funding, legal).")

    return {"score": score, "notes": notes}


def step5_policy_and_monitoring(data):
    """
    Step 5: CREATE adaptive_policy_framework, INCLUDE review_boards, ESTABLISH monitoring_mechanisms
    """
    policy_framework = data.get("policy_framework", False)
    review_board = data.get("review_board", False)
    monitoring_plan = data.get("monitoring_plan", False)
    notes = []
    score = 0.0

    score += bool_score(policy_framework, weight=0.4)
    score += bool_score(review_board, weight=0.3)
    score += bool_score(monitoring_plan, weight=0.3)

    if not policy_framework:
        notes.append("Adaptive policy framework absent.")
    if not review_board:
        notes.append("Independent review/ethics board not identified.")
    if not monitoring_plan:
        notes.append("Long-term ecological monitoring plan missing.")

    return {"score": score, "notes": notes}


def step6_coordination_and_enforcement(data):
    """
    Step 6: COORDINATE with national_and_international_agencies, DEFINE roles, ENFORCE penalties
    """
    coordination = data.get("international_coordination", False)
    roles_defined = data.get("roles_defined", False)
    enforcement = data.get("enforcement_mechanisms", False)
    notes = []
    score = 0.0

    score += bool_score(coordination, weight=0.4)
    score += bool_score(roles_defined, weight=0.3)
    score += bool_score(enforcement, weight=0.3)

    if not coordination:
        notes.append("No regional/international coordination plan.")
    if not roles_defined:
        notes.append("Implementation roles not clearly assigned.")
    if not enforcement:
        notes.append("Penalties or legal enforcement unclear.")

    return {"score": score, "notes": notes}


def step7_monitor_and_review(data):
    """
    Step 7: MONITOR ecological_outcomes, REVIEW framework periodically, UPDATE policies
    """
    monitoring_active = data.get("monitoring_active", False)
    review_schedule = data.get("review_schedule_years", None)
    update_policy_on_data = data.get("update_policy_on_data", False)
    notes = []
    score = 0.0

    score += bool_score(monitoring_active, weight=0.5)
    if isinstance(review_schedule, (int, float)) and review_schedule <= 5:
        # frequent reviews are better
        score += 30.0
    elif isinstance(review_schedule, (int, float)):
        score += max(0.0, 30.0 - min(20.0, review_schedule * 2.0))
    else:
        notes.append("No review schedule given.")

    score += bool_score(update_policy_on_data, weight=0.2)

    if not monitoring_active:
        notes.append("Monitoring not active or not funded.")
    if not update_policy_on_data:
        notes.append("Policy update trigger on new data not defined.")

    return {"score": score, "notes": notes}


# --- Orchestration ---


def run_rbra(data):
    """Run the full 7-step RBRA and produce a summarized report."""
    results = {}
    total_weight = 0.0
    # define weights for overall averaging (these are heuristic)
    step_weights = {
        "step1": 1.0,
        "step2": 1.0,
        "step3": 1.2,
        "step4": 1.5,
        "step5": 1.0,
        "step6": 0.8,
        "step7": 1.0,
    }

    s1 = step1_scope_and_objectives(data)
    s2 = step2_collect_scientific_data(data)
    s3 = step3_analyze_risks(data)
    s4 = step4_mitigation_and_kill_switch(data)
    s5 = step5_policy_and_monitoring(data)
    s6 = step6_coordination_and_enforcement(data)
    s7 = step7_monitor_and_review(data)

    results["step1"] = s1
    results["step2"] = s2
    results["step3"] = s3
    results["step4"] = s4
    results["step5"] = s5
    results["step6"] = s6
    results["step7"] = s7

    # Weighted average
    weighted_sum = (
        s1["score"] * step_weights["step1"]
        + s2["score"] * step_weights["step2"]
        + s3["score"] * step_weights["step3"]
        + s4["score"] * step_weights["step4"]
        + s5["score"] * step_weights["step5"]
        + s6["score"] * step_weights["step6"]
        + s7["score"] * step_weights["step7"]
    )
    total_weight = sum(step_weights.values())
    overall_score = weighted_sum / total_weight

    # readiness bands
    if overall_score >= 85:
        readiness = "High — ready for strictly controlled field trials with approvals"
    elif overall_score >= 60:
        readiness = "Moderate — work remaining; do not release until gaps closed"
    elif overall_score >= 40:
        readiness = "Low — significant missing elements, revise and re-submit"
    else:
        readiness = "Very Low — do not proceed. Reassess basic necessity and safety."

    results["overall_score"] = round(overall_score, 1)
    results["readiness"] = readiness

    # collect all notes into a single list (deduplicated)
    notes = []
    for k in ["step1", "step2", "step3", "step4", "step5", "step6", "step7"]:
        for n in results[k]["notes"]:
            if n not in notes:
                notes.append(n)
    results["issues"] = notes
    results["recommendations"] = generate_recommendations(results)

    return results


def generate_recommendations(results):
    """Generate plain-language recommendations based on findings."""
    recs = []
    score = results["overall_score"]

    if score < 40:
        recs.append("Re-evaluate whether gene editing is necessary. Consider non-GE alternatives.")
    if "No 'kill-switch' or reversible control measures defined." in results["issues"]:
        recs.append("Design and test a robust reversible control / kill-switch prior to any environmental release.")
    if "Incomplete scientific data (e.g., proof of mechanism, lab/containment results)." in results["issues"]:
        recs.append("Complete laboratory and contained field data demonstrating mechanism of action and confinement.")
    if "No comparison with existing national/international rules." in results["issues"]:
        recs.append("Perform a regulatory scan: identify relevant national rules and international guidance (WHO, CBD, National Academies).")
    if "Long-term ecological monitoring plan missing." in results["issues"]:
        recs.append("Create a funded, multi-year ecological monitoring plan with clear indicators and sampling protocols.")
    if "No regional/international coordination plan." in results["issues"]:
        recs.append("Engage neighboring jurisdictions and international bodies; define transboundary response protocols.")
    if not recs:
        recs.append("Project appears well-prepared: proceed to formal review boards and controlled field trial planning.")

    # always include a short policy template
    recs.append(create_policy_template(results))

    return recs


def create_policy_template(results):
    """Produce a short adaptive policy & monitoring template string."""
    template = [
        "Adaptive Policy & Monitoring Template:",
        "- Objective: " + ("Demonstrated necessity and conservation objective." if results["step1"]["score"] > 50 else "Objective needs clearer justification."),
        "- Independent Review: Establish an independent ethics and risk review board with local and international members.",
        "- Mitigation & Emergency Plan: Publish detailed mitigation measures and a tested 'kill-switch' procedure with assigned responsible parties.",
        "- Monitoring: Funded monitoring for at least 10 years, defined KPIs, baseline data, and public reporting schedule.",
        "- Coordination: Signed agreement/MoU with neighboring jurisdictions to manage transboundary risks and shared responsibilities.",
        "- Review Cycle: Policy and permits reviewed every 1–3 years or sooner if monitoring data indicate unexpected outcomes.",
    ]
    return "\n".join(template)


def pretty_print_results(results):
    print("\nRBRA Report")
    print("-----------")
    print(f"Overall readiness score: {results['overall_score']} / 100")
    print(f"Readiness level: {results['readiness']}\n")
    print("Step scores (0-100):")
    for i in range(1, 8):
        step = f"step{i}"
        print(f"  Step {i}: {results[step]['score']:.1f}")
        if results[step]["notes"]:
            for n in results[step]["notes"]:
                print(f"    - {n}")
    print("\nIssues found:")
    if results["issues"]:
        for n in results["issues"]:
            print(" - " + n)
    else:
        print(" None.")

    print("\nRecommendations:")
    for r in results["recommendations"]:
        if isinstance(r, str) and r.startswith("Adaptive Policy"):
            print("\n" + r + "\n")
        else:
            print(" - " + r)


# --- Example / CLI ---


EXAMPLE_DATA = {
    # Step 1
    "alternatives_assessed": True,
    "necessity_evidence": True,
    # Step 2
    "scientific_data_complete": False,  # suppose lab trials not fully complete
    "regulatory_scan": False,
    "stakeholder_engagement": True,
    # Step 3 (0..10 where higher is more risk)
    "ecological_risk": 6.5,
    "ethical_risk": 4.0,
    "socioeconomic_risk": 5.0,
    # Step 4
    "mitigation_plan": True,
    "kill_switch": False,
    "response_capacity": False,
    # Step 5
    "policy_framework": False,
    "review_board": True,
    "monitoring_plan": False,
    # Step 6
    "international_coordination": False,
    "roles_defined": False,
    "enforcement_mechanisms": False,
    # Step 7
    "monitoring_active": False,
    "review_schedule_years": None,
    "update_policy_on_data": False,
}


def main():
    parser = argparse.ArgumentParser(description="Run RBRA interactive evaluation")
    parser.add_argument("--example", action="store_true", help="Run example dataset")
    args = parser.parse_args()

    if args.example:
        data = EXAMPLE_DATA
        print("Running example dataset...\n")
        results = run_rbra(data)
        pretty_print_results(results)
        return

    # Interactive prompts
    print("RBRA interactive tool — answer prompts (y/n or number). Press Enter to accept default shown in []\n")

    data = {}

    def ask_bool(prompt, default=False):
        d = "y" if default else "n"
        val = input(f"{prompt} [default {d}]: ").strip().lower()
        if val == "":
            return default
        return val in ("y", "yes", "1")

    def ask_float(prompt, default=5.0):
        val = input(f"{prompt} [default {default}]: ").strip()
        if val == "":
            return default
        try:
            return float(val)
        except ValueError:
            print("Invalid number, using default.")
            return default

    # Step 1
    data["alternatives_assessed"] = ask_bool("Step1: Have non-GE alternatives been fully assessed? (y/n)", False)
    data["necessity_evidence"] = ask_bool("Step1: Is there documented evidence that alternatives failed and GE is necessary? (y/n)", False)

    # Step 2
    data["scientific_data_complete"] = ask_bool("Step2: Is the scientific dossier (lab, modelling, containment) complete? (y/n)", False)
    data["regulatory_scan"] = ask_bool("Step2: Has a regulatory scan (national/international) been performed? (y/n)", False)
    data["stakeholder_engagement"] = ask_bool("Step2: Have stakeholders (local communities, indigenous groups) been consulted? (y/n)", False)

    # Step 3
    print("Step3 risk inputs (0 = no risk, 10 = very high risk).")
    data["ecological_risk"] = ask_float("Ecological risk (0-10)?", 5.0)
    data["ethical_risk"] = ask_float("Ethical risk (0-10)?", 5.0)
    data["socioeconomic_risk"] = ask_float("Socioeconomic impact risk (0-10)?", 5.0)

    # Step 4
    data["mitigation_plan"] = ask_bool("Step4: Is there a published mitigation plan? (y/n)", False)
    data["kill_switch"] = ask_bool("Step4: Is there a tested kill-switch or reversal method? (y/n)", False)
    data["response_capacity"] = ask_bool("Step4: Is emergency response capacity (labs, funds, legal) in place? (y/n)", False)

    # Step 5
    data["policy_framework"] = ask_bool("Step5: Is an adaptive policy framework defined? (y/n)", False)
    data["review_board"] = ask_bool("Step5: Is an independent review board identified? (y/n)", False)
    data["monitoring_plan"] = ask_bool("Step5: Is a long-term monitoring plan prepared? (y/n)", False)

    # Step 6
    data["international_coordination"] = ask_bool("Step6: Is there international/regional coordination planned? (y/n)", False)
    data["roles_defined"] = ask_bool("Step6: Are roles and responsibilities clearly assigned? (y/n)", False)
    data["enforcement_mechanisms"] = ask_bool("Step6: Are legal enforcement and penalties defined? (y/n)", False)

    # Step 7
    data["monitoring_active"] = ask_bool("Step7: Is monitoring actively funded and ongoing? (y/n)", False)
    rs = input("Step7: Review schedule (years). Enter a number (e.g., 1, 3, 5) or leave blank: ").strip()
    try:
        data["review_schedule_years"] = int(rs) if rs else None
    except ValueError:
        data["review_schedule_years"] = None
    data["update_policy_on_data"] = ask_bool("Step7: Will policy be updated automatically on new data triggers? (y/n)", False)

    print("\nRunning RBRA analysis...\n")
    results = run_rbra(data)
    pretty_print_results(results)

    # Optionally dump json
    save = input("\nSave report to rbra_report.json? (y/n) [n]: ").strip().lower()
    if save in ("y", "yes"):
        with open("rbra_report.json", "w") as f:
            json.dump(results, f, indent=2)
        print("Saved report to rbra_report.json")


if __name__ == "__main__":
    main()
