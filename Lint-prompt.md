You are reviewing Swift code changes in the CURRENT BRANCH ONLY.

Your job is to ensure that this branch does NOT introduce any new SwiftLint violations.

IMPORTANT:
- Fix ONLY lint violations that are NEWLY INTRODUCED by this branch.
- DO NOT modify or fix any pre-existing lint violations that were already in the base branch.
- DO NOT modify any lines that were not changed in this branch.
- Always choose the minimal fix that satisfies the rule.

====================================================================
# 1 — SwiftLint Configuration (Primary Lint Rules)

only_rules:
  - anyobject_protocol
  - closing_brace
  - closure_end_indentation
  - closure_spacing
  - colon
  - comma
  - contains_over_filter_count
  - contains_over_filter_is_empty
  - contains_over_first_not_nil
  - contains_over_range_nil_comparison
  - control_statement
  - convenience_type
  - custom_rules
  - duplicated_key_in_dictionary_literal
  - empty_count
  - empty_enum_arguments
  - empty_parentheses_with_trailing_closure
  - explicit_init
  - fatal_error_message
  - file_length
  - force_cast
  - force_try
  - force_unwrapping
  - identifier_name
  - implicit_getter
  - implicit_return
  - implicitly_unwrapped_optional
  - legacy_cggeometry_functions
  - legacy_constant
  - legacy_constructor
  - legacy_nsgeometry_functions
  - legacy_objc_type
  - line_length
  - literal_expression_end_indentation
  - mark
  - multiline_arguments
  - multiline_arguments_brackets
  - multiline_function_chains
  - multiline_literal_brackets
  - multiline_parameters
  - multiline_parameters_brackets
  - no_grouping_extension
  - no_space_in_method_call
  - operator_usage_whitespace
  - overridden_super_call
  - prefer_zero_over_explicit_init
  - private_outlet
  - private_over_fileprivate
  - redundant_optional_initialization
  - redundant_string_enum_value
  - redundant_void_return
  - return_arrow_whitespace
  - strong_iboutlet
  - syntactic_sugar
  - toggle_bool
  - trailing_closure
  - trailing_newline
  - trailing_whitespace
  - type_name
  - unneeded_parentheses_in_closure_argument
  - unused_closure_parameter
  - unused_import
  - unused_optional_binding
  - vertical_whitespace
  - void_return
  - weak_delegate
  - comment_spacing
  - computed_accessors_order
  - duplicate_enum_cases
  - duplicate_imports
  - opening_brace
  - trailing_comma
  - trailing_semicolon
  - unneeded_break_in_switch

excluded:
  - Carthage
  - Pods
  - SwaggerClient/SwaggerClient/Source/Legacy/Classes/Swagger/Models
  - XcodeTemplates
  - scripts
  - fastlane
  - GraphQLClient/Sources/GraphQLGeneratedSources

line_length:
  warning: 120

file_length:
  warning: 600

identifier_name:
  max_length:
    warning: 50
    error: 60
  excluded:
    - id
    - ID
    - x
    - y
    - dx
    - dy
    - to
    - up

type_name:
  allowed_symbols: "_"
  max_length:
    warning: 50
    error: 60
  excluded:
    - Id

indentation: 4

custom_rules:
  no_objcMembers:
    name: "@objcMembers"
    regex: "@objcMembers"
    message: "Explicitly use @objc on each member you want to expose to Objective-C"
    severity: error

  disable_print:
    included: ".*\\.swift"
    name: "print usage"
    regex: "((\\bprint)|(Swift\\.print))\\s*\\("
    message: "print is not allowed due to PCI compliance and security reasons. Use LoggingManagement API."
    severity: error

  disable_debugPrint:
    included: ".*\\.swift"
    name: "debugPrint usage"
    regex: "((\\bdebugPrint)|(Swift\\.debugPrint))\\s*\\("
    message: "debugPrint is not allowed. Use LoggingManagement."
    severity: error

  disable_nslog:
    included: ".*\\.swift"
    name: "NSLog usage"
    regex: "((\\bNSLog))\\s*\\("
    message: "NSLog is not allowed. Use LoggingManagement."
    severity: error

  enhanced_assertion:
    included: ".*\\.swift"
    excluded: ".*Tests\\.swift"
    name: "Prefer `Assert` over `assert`."
    regex: "\\sassert(|ionFailure)\\("
    message: "Use Assert methods so failures are caught in production."
    severity: warning
    match_kinds:
      - identifier

  explicit_init_without_type:
    included: ".*\\.swift"
    name: "Explicit .init() without type."
    regex: "(?<!\\b(super|self))\\.init\\("
    message: "Object should be instantiated with explicit type."
    severity: warning

====================================================================
# 2 — Auto-correct Formatting Rules (Secondary Fix Pass)

only_rules:
  - comment_spacing
  - closing_brace
  - closure_end_indentation
  - closure_spacing
  - colon
  - comma
  - empty_enum_arguments
  - empty_parameters
  - empty_parentheses_with_trailing_closure
  - implicit_return
  - leading_whitespace
  - literal_expression_end_indentation
  - mark
  - modifier_order
  - no_space_in_method_call
  - opening_brace
  - redundant_void_return
  - return_arrow_whitespace
  - sorted_imports
  - trailing_comma
  - trailing_newline
  - trailing_semicolon
  - trailing_whitespace
  - unneeded_parentheses_in_closure_argument
  - unused_closure_parameter
  - unused_import
  - vertical_whitespace
  - vertical_whitespace_closing_braces
  - void_return

====================================================================
# 3 — Critical Logic: Fix ONLY New Violations

You MUST:

1. Compare the changed lines against the base version (represented in the diff).
2. Fix *only* violations introduced or modified by this branch.
3. Do NOT fix or touch any violations present in unchanged lines.
4. Do NOT rewrite or refactor code unless needed to resolve a lint rule.
5. Make minimal, targeted fixes.

====================================================================
# 4 — Required Output Format

### SECTION A — Summary
- Number of new violations introduced  
- Number fixed  
- Any ambiguous cases that may require human review  

### SECTION B — Corrected Code
- Provide corrected versions ONLY for the modified code hunks

### SECTION C — New Violation Audit
For EACH new violation this branch introduced:
- Rule name  
- Original code snippet  
- Corrected code snippet  
- Explanation  

### SECTION D — Final Confirmation
State exactly:

**“All newly introduced SwiftLint violations have been fixed. No pre-existing violations were modified.”**

====================================================================

Below are the branch changes to analyze:
<INSERT DIFF OR UPDATED FILE CONTENTS HERE>
