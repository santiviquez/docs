---
layout: default
title: Reference checks
description: Use a SodaCL reference check to validate that the values in a column in a table are present in a column in a different table. 
parent: SodaCL reference
---

# Reference checks 
<!--Linked to UI, access Shlink-->
*Last modified on {% last_modified_at %}*

Use a reference check to validate that column contents match between datasets in the same data source. 

See also: [Compare data using SodaCL]({% link soda-cl/compare.md %})
{% include code-header.html %}
```yaml
checks for dim_department_group:
  - values in (department_group_name) must exist in dim_employee (department_name)
  - values in (birthdate) must not exist in dim_department_group_prod (birthdate)
```
<small>✖️ &nbsp;&nbsp; Requires Soda Core Scientific (included in a Soda Agent)</small><br />
<small>✔️ &nbsp;&nbsp; Supported in Soda Core</small><br />
<small>✔️ &nbsp;&nbsp; Supported in Soda Library + Soda Cloud</small><br />
<small>✔️ &nbsp;&nbsp; Supported in Soda Cloud Agreements + Soda Agent</small><br />
<small>✖️ &nbsp;&nbsp; Available as a no-code check</small>
<br />

[Define reference checks](#define-reference-checks) <br />
&nbsp;&nbsp;&nbsp;&nbsp;[Failed row samples](#failed-row-samples)<br />
[Optional check configurations](#optional-check-configurations)<br />
[Go further](#go-further)<br />
<br />


## Define reference checks

In the context of [SodaCL check types]({% link soda-cl/metrics-and-checks.md %}#check-types), reference checks are unique. This check is limited in its syntax variation, with only a few mutable parts to specify column and dataset names.

The example below checks that the values in the source column, `department_group_name`, in the `dim_department_group` dataset exist somewhere in the destination column, `department_name`, in the `dim_employee` dataset. If the values are absent in the `department_name` column, the check fails.
* Soda CL considers missing values in the source column as invalid.
* Optionally, do not use brackets around column names. The brackets serve as visual aids to improve check readability.
{% include code-header.html %}
```yaml
checks for dim_department_group:
  - values in (department_group_name) must exist in dim_employee (department_name)
```

You can also validate that data in one dataset does *not* exist in another.  
{% include code-header.html %}
```yaml
checks for dim_customer_staging:
  - values in (birthdate) must not exist in dim_customer_prod (birthdate)
```

### Reference checks and dataframes

{% include reference-with-spark.md %}

### Failed row samples

Reference checks automatically collect samples of any failed rows to display Soda Cloud. The default number of failed row samples that Soda collects and displays is 100.

If you wish to limit or broaden the sample size, you can use the `samples limit` configuration in a reference check configuration. You can add this configuration to your checks YAML file for Soda Library, or when writing checks as part of an agreement in Soda Cloud. See: [Set a sample limit]({% link soda-cl/failed-row-samples.md %}#set-a-sample-limit).
{% include code-header.html %}
```yaml
checks for dim_customers:
  - values in (state_code, state_name) must exist in iso_3166-2 (code, subdivision_name):
      samples limit: 20
``` 
<br />

For security, you can add a configuration to your data source connection details to prevent Soda from collecting failed rows samples from specific columns that contain sensitive data. See: [Disable failed row samples]({% link soda-cl/failed-row-samples.md %}#disable-failed-row-samples).

Alternatively, you can set the `samples limit` to `0` to prevent Soda from collecting and sending failed rows samples for an individual check, as in the following example.
{% include code-header.html %}
```yaml
checks for dim_customers:
  - values in (state_code, state_name) must exist in iso_3166-2 (code, subdivision_name):
      samples limit: 0
``` 
<br />

You can also use a `samples columns` or a `collect failed rows` configuration to a check to specify the columns for which Soda must implicitly collect failed row sample values, as in the following example with the former. Soda only collects this check's failed row samples for the columns you specify in the list. See: [Customize sampling for checks]({% link soda-cl/failed-row-samples.md %}#customize-sampling-for-checks).

Note that the comma-separated list of samples columns does not support wildcard characters (%).
```yaml
checks for dim_customers:
  - values in (state_code, state_name) must exist in iso_3166-2 (code, subdivision_name):
      samples columns: [state_code]
```
<br />

To review the failed rows in Soda Cloud, navigate to the **Checks** dashboard, then click the row for a reference check. Examine failed rows in the **Failed Rows Analysis** tab; see [Manage failed row samples]({% link soda-cl/failed-row-samples.md %}) for further details.


## Optional check configurations

| Supported | Configuration | Documentation |
| :-: | ------------|---------------|
| ✓ | Define a name for a reference check; see [example](#example-with-check-name). |  [Customize check names]({% link soda-cl/optional-config.md %}#customize-check-names) |
| ✓ | Add an identity to a check. | [Add a check identity]({% link soda-cl/optional-config.md %}#add-a-check-identity) |
|   | Define alert configurations to specify warn and fail alert conditions. | - |
|   | Apply an in-check filter to return results for a specific portion of the data in your dataset.| - | 
| ✓ | Use quotes when identifying dataset or column names; see [example](#example-with-quotes). <br />Note that the type of quotes you use must match that which your data source uses. For example, BigQuery uses a backtick ({% raw %}`{% endraw %}) as a quotation mark. | [Use quotes in a check]({% link soda-cl/optional-config.md %}#use-quotes-in-a-check) |
|   | Use wildcard characters ({% raw %} % {% endraw %} or {% raw %} * {% endraw %}) in values in the check. | - |
|   | Use for each to apply reference checks to multiple datasets in one scan. | - |
| ✓ | Apply a dataset filter to partition data during a scan; see [example](#example-with-dataset-filter). If you encounter difficulties, see [Filter not passed with reference check]({% link soda-cl/troubleshoot.md %}#filter-not-passed-with-reference-check). | [Scan a portion of your dataset]({% link soda-cl/optional-config.md %}#scan-a-portion-of-your-dataset) |
| ✓ | Supports `samples columns` parameter to specify columns from which Soda draws failed row samples. | [Customize sampling for checks]({% link soda-cl/failed-row-samples.md %}#customize-sampling-for-checks) |
| ✓ | Supports `samples limit` parameter to control the volume of failed row samples Soda collects. | [Set a sample limit]({% link soda-cl/failed-row-samples.md %}#set-a-sample-limit) |
| ✓ | Supports `collect failed rows` parameter instruct Soda to collect, or not to collect, failed row samples for a check. | [Customize sampling for checks]({% link soda-cl/failed-row-samples.md %}#customize-sampling-for-checks) |

#### Example with check name 
{% include code-header.html %}
```yaml
checks for dim_department_group:
  - values in (department_group_name) must exist in dim_employee (department_name):
      name: Compare department datasets
```

#### Example with quotes
{% include code-header.html %}
```yaml
checks for dim_department_group:
  - values in ("department_group_name") must exist in dim_employee ("department_name")
```

#### Example with dataset filter

Refer to [Troubleshoot SodaCL]({% link soda-cl/troubleshoot.md %}#filter-not-passed-with-reference-check) to address challenges specific to reference checks with dataset filters.

{% include code-header.html %}
```yaml
filter customers_c8d90f60 [daily]:
  where: ts > TIMESTAMP '${NOW}' - interval '100y'

checks for customers_c8d90f60 [daily]:
  - values in (cat) must exist in customers_europe (cat2)
```



<br />

## Go further

* Problems with reference checks and dataset filters? Refer to [Troubleshoot SodaCL]({% link soda-cl/troubleshoot.md %}#filter-not-passed-with-reference-check).
* Learn more about [SodaCL metrics and checks]({% link soda-cl/metrics-and-checks.md %}) in general.
* Learn more about [comparing data]({% link soda-cl/compare.md %}) using SodaCL.
* Use a [schema check]({% link soda-cl/schema.md %}) to discover missing or forbidden columns in a dataset.
* Need help? Join the <a href="https://community.soda.io/slack" target="_blank"> Soda community on Slack</a>.
* Reference [tips and best practices for SodaCL]({% link soda/quick-start-sodacl.md %}#tips-and-best-practices-for-sodacl).
<br />


---

Was this documentation helpful?

<!-- LikeBtn.com BEGIN -->
<span class="likebtn-wrapper" data-theme="tick" data-i18n_like="Yes" data-ef_voting="grow" data-show_dislike_label="true" data-counter_zero_show="true" data-i18n_dislike="No"></span>
<script>(function(d,e,s){if(d.getElementById("likebtn_wjs"))return;a=d.createElement(e);m=d.getElementsByTagName(e)[0];a.async=1;a.id="likebtn_wjs";a.src=s;m.parentNode.insertBefore(a, m)})(document,"script","//w.likebtn.com/js/w/widget.js");</script>
<!-- LikeBtn.com END -->

{% include docs-footer.md %}
