---
output: github_document
---

<!-- README.md is generated from README.Rmd. Please edit that file -->



# PsyR

<!-- badges: start -->
<!-- badges: end -->

PsyR computes confidence intervals for planned and post-hoc contrasts
from ANOVA models, including procedures described by Bird (2002) for
psychology research.

## Installation

You can install the development version of PsyR from [GitHub](https://github.com/) with:


``` r
# install.packages("devtools")
devtools::install_github("garner-code/PsyR")
```

## Example

This is a basic example which shows you how to obtain between subject confidence
intervals. All of the following examples will use the post-hoc method
(see Bird, 2002, for details on the post-hoc method):


``` r

## load relevant packages
library(emmeans)
library(afex)
library(PsyR)

# load some data
dat <- spacing

# set emmeans option to multivariate
afex_options(emmeans_model = "multivariate")

# perform the statistical model
mod <- aov_ez("subj", "yield", dat, within = "spacing", between = "group")

# define and mean-difference scale some between-group contrasts
emm_btwn <- emmeans(mod, "group")
raw_con_b <- list(
  "12vs34" = c(1, 1, -1, -1), # groups 1 & 2 vs groups 3 & 4
  "1vs2" = c(1, -1, 0, 0), # and so on...
  "3vs4" = c(0, 0, 1, -1)
)
con_b <- rescale_contrasts(raw_con_b, mode = "mean_difference")
btwn_con <- contrast(emm_btwn, con_b)

# feed the contrast table into psyci(); its family and participating factors
# are inferred from the model and the original emmGrid
psyci(model = mod, contrast_tables = list(btwn_con), method = "ph")
#> Raw contrast effects and confidence intervals (dependent-variable units)
#>
#> $b
#>   contrast   estimate       SE df_error    F_value psyr_p_value       cc
#> 1   12vs34   2.958333 2.115720       12  1.9551390  0.596976752 3.235875
#> 2     1vs2 -14.083333 2.992081       12 22.1546218  0.004609298 3.235875
#> 3     3vs4  -2.500000 2.992081       12  0.6981254  0.871845486 3.235875
#>        lower     upper
#> 1  -3.887873  9.804540
#> 2 -23.765331 -4.401336
#> 3 -12.181998  7.181998
#>
#> Results are averaged over the levels of: spacing
#> PsyR CI method: Scheffe has been applied
#> PsyR p-value method: Scheffe has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: between subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed between subject factor(s) are: group
#> PsyR used df between of: 3
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality with other tables was not applicable because only one table was supplied.
#>
#> Scaled contrast effects and confidence intervals (sample SD units)
#> Scaling divisor (pooled sample SD): 4.625563
#>
#> $b
#>   contrast   estimate        SE df_error    F_value psyr_p_value       cc
#> 1   12vs34  0.6395618 0.4573974       12  1.9551390  0.596976752 3.235875
#> 2     1vs2 -3.0446744 0.6468576       12 22.1546218  0.004609298 3.235875
#> 3     3vs4 -0.5404747 0.6468576       12  0.6981254  0.871845486 3.235875
#>        lower      upper
#> 1 -0.8405188  2.1196424
#> 2 -5.1378245 -0.9515243
#> 3 -2.6336248  1.5526753
#>
#> Results are averaged over the levels of: spacing
#> PsyR CI method: Scheffe has been applied
#> PsyR p-value method: Scheffe has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: between subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed between subject factor(s) are: group
#> PsyR used df between of: 3
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality with other tables was not applicable because only one table was supplied.
#> PsyR scaled estimate, SE, and confidence limits by sample SD: 4.625563
```

`psyci()` prints each result twice. The first table reports the contrast
effects and confidence intervals in the dependent variable's original units.
The second reports the same values in sample standard-deviation units, using
the pooled within-cell scaling method from the original Psy program. The raw
tables remain available through ordinary list indexing. For example:


``` r
result <- psyci(mod, list(btwn_con), method = "ph")
raw_table <- result[[1]]
scaled_table <- attr(result, "scaled_tables")[[1]]
attr(result, "scale_info")$divisor
```

The standard tables report `df_error`, Psy's observed one-degree-of-freedom
`F_value`, and the correction-specific `psyr_p_value`. The original `emmeans`
`t.ratio` and `p.value` columns are not retained. To print Psy's legacy
SS/df/MS/F layout for the supplied contrasts and keep it with the result, use:


``` r
result <- psyci(
  mod,
  list(btwn_con),
  method = "ph",
  legacy_anova = TRUE
)
legacy_table <- attr(result, "legacy_anova_table")
raw_table[["df_error"]]
```

The legacy table is printed once, before the raw and standardized confidence
interval tables. It includes only supplied contrasts and their applicable error
rows; it does not create absent between-by-within cross-products.

Here is an example, that carries on from the last, for obtaining within subject
confidence intervals:


``` r

# define and mean-difference scale some within-subject contrasts
emm_win <- emmeans(mod, "spacing")
raw_con_w <- list(
  "20vs40" = c(1, -1, 0),
  "20vs60" = c(1, 0, -1),
  "Quad" = c(1, -2, 1)
)
con_w <- rescale_contrasts(raw_con_w, mode = "mean_difference")
con_win <- contrast(emm_win, con_w)

# generate 95% CIs for the within subjects contrasts
psyci(model=mod, contrast_tables = list(con_win), method="ph")
#> Raw contrast effects and confidence intervals (dependent-variable units)
#>
#> $w
#>   contrast estimate       SE df_error   F_value psyr_p_value       cc     lower
#> 1   20vs40    -1.75 0.832291       12 4.4210526   0.17815228 2.947652 -4.203304
#> 2   20vs60    -3.00 1.014222       12 8.7493671   0.04920057 2.947652 -5.989572
#> 3     Quad    -0.25 0.458570       12 0.2972136   0.87409756 2.947652 -1.601705
#>        upper
#> 1  0.7033040
#> 2 -0.0104275
#> 3  1.1017046
#>
#> Results are averaged over the levels of: group
#> PsyR CI method: post-hoc within (Hotelling's T-squared) has been applied
#> PsyR p-value method: post-hoc within (Hotelling's T-squared) has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: within subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed within subject factor(s) are: spacing
#> PsyR used df within of: 2
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality with other tables was not applicable because only one table was supplied.
#>
#> Scaled contrast effects and confidence intervals (sample SD units)
#> Scaling divisor (pooled sample SD): 4.625563
#>
#> $w
#>   contrast    estimate        SE df_error   F_value psyr_p_value       cc
#> 1   20vs40 -0.37833232 0.1799329       12 4.4210526   0.17815228 2.947652
#> 2   20vs60 -0.64856969 0.2192645       12 8.7493671   0.04920057 2.947652
#> 3     Quad -0.05404747 0.0991382       12 0.2972136   0.87409756 2.947652
#>        lower        upper
#> 1 -0.9087119  0.152047212
#> 2 -1.2948851 -0.002254321
#> 3 -0.3462724  0.238177416
#>
#> Results are averaged over the levels of: group
#> PsyR CI method: post-hoc within (Hotelling's T-squared) has been applied
#> PsyR p-value method: post-hoc within (Hotelling's T-squared) has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: within subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed within subject factor(s) are: spacing
#> PsyR used df within of: 2
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality with other tables was not applicable because only one table was supplied.
#> PsyR scaled estimate, SE, and confidence limits by sample SD: 4.625563
```

For pairwise correlated within-subject means, PsyR also provides the asymmetric
noncentral-*t* standardized-effect interval proposed by Algina and Keselman
(2003). It is opt-in, so existing calls retain the original Psy/Bird output.
Use it with individual or Bonferroni intervals:


``` r
result_ak <- psyci(
  mod,
  list(con_win),
  method = "bf",
  standardized_ci = "algina_keselman"
)
ak_table <- attr(result_ak, "scaled_tables")[[1]]
```

The new `algina_keselman_ci()` function also exposes the calculation directly
when an estimated contrast, its standard error, error degrees of freedom, and
the appropriate sample-SD scaling divisor are already available.

And finally, here is an example of how to generate CIs for between x within
contrasts, using the same post-hoc method:


``` r

# get emms for each cell from the between x within design
emm_int <- emmeans(mod, c("group", "spacing"))

# emmeans forms products of the supplied between and within coefficients. It
# does not normalize those coefficients. Because con_b and con_w were each
# scaled above as order-0 mean differences, their products are correctly scaled
# order-1 interactions (differences between mean differences).

con_int <- contrast(emm_int, interaction=list(con_b, con_w))
# generate 95% CIs for the between x within subjects contrasts
psyci(model=mod, contrast_tables = list(con_int), method="ph")
#> Raw contrast effects and confidence intervals (dependent-variable units)
#>
#> $bw
#>   group_custom spacing_custom estimate       SE df_error     F_value
#> 1       12vs34         20vs40   -8.000 1.664582       12 23.09774436
#> 2         1vs2         20vs40    5.000 2.354074       12  4.51127820
#> 3         3vs4         20vs40   -3.000 2.354074       12  1.62406015
#> 4       12vs34         20vs60  -19.250 2.028444       12 90.06075949
#> 5         1vs2         20vs60    8.750 2.868652       12  9.30379747
#> 6         3vs4         20vs60   -6.750 2.868652       12  5.53670886
#> 7       12vs34           Quad    1.625 0.917140       12  3.13931889
#> 8         1vs2           Quad    0.625 1.297032       12  0.23219814
#> 9         3vs4           Quad    0.375 1.297032       12  0.08359133
#>   psyr_p_value       cc      lower      upper
#> 1 2.157666e-02 4.191986 -14.977905  -1.022095
#> 2 5.884826e-01 4.191986  -4.868248  14.868248
#> 3 9.297047e-01 4.191986 -12.868248   6.868248
#> 4 8.158899e-05 4.191986 -27.753208 -10.746792
#> 5 2.240525e-01 4.191986  -3.275351  20.775351
#> 6 4.807223e-01 4.191986 -18.775351   5.275351
#> 7 7.546456e-01 4.191986  -2.219638   5.469638
#> 8 9.995511e-01 4.191986  -4.812140   6.062140
#> 9 9.999771e-01 4.191986  -5.062140   5.812140
#>
#> PsyR CI method: post-hoc between x within (Roy's GCR) has been applied
#> PsyR p-value method: post-hoc between x within (Roy's GCR) has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: between x within subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed between subject factor(s) are: group
#> PsyR used df between of: 3
#> PsyR assumed within subject factor(s) are: spacing
#> PsyR used df within of: 2
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality with other tables was not applicable because only one table was supplied.
#>
#> Scaled contrast effects and confidence intervals (sample SD units)
#> Scaling divisor (pooled sample SD): 4.625563
#>
#> $bw
#>   group_custom spacing_custom    estimate        SE df_error     F_value
#> 1       12vs34         20vs40 -1.72951919 0.3598658       12 23.09774436
#> 2         1vs2         20vs40  1.08094949 0.5089271       12  4.51127820
#> 3         3vs4         20vs40 -0.64856969 0.5089271       12  1.62406015
#> 4       12vs34         20vs60 -4.16165554 0.4385290       12 90.06075949
#> 5         1vs2         20vs60  1.89166161 0.6201737       12  9.30379747
#> 6         3vs4         20vs60 -1.45928181 0.6201737       12  5.53670886
#> 7       12vs34           Quad  0.35130858 0.1982764       12  3.13931889
#> 8         1vs2           Quad  0.13511869 0.2804052       12  0.23219814
#> 9         3vs4           Quad  0.08107121 0.2804052       12  0.08359133
#>   psyr_p_value       cc      lower      upper
#> 1 2.157666e-02 4.191986 -3.2380717 -0.2209666
#> 2 5.884826e-01 4.191986 -1.0524660  3.2143650
#> 3 9.297047e-01 4.191986 -2.7819852  1.4848458
#> 4 8.158899e-05 4.191986 -5.9999631 -2.3233480
#> 5 2.240525e-01 4.191986 -0.7080979  4.4914211
#> 6 4.807223e-01 4.191986 -4.0590413  1.1404777
#> 7 7.546456e-01 4.191986 -0.4798634  1.1824806
#> 8 9.995511e-01 4.191986 -1.0403360  1.3105734
#> 9 9.999771e-01 4.191986 -1.0943835  1.2565259
#>
#> PsyR CI method: post-hoc between x within (Roy's GCR) has been applied
#> PsyR p-value method: post-hoc between x within (Roy's GCR) has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: between x within subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed between subject factor(s) are: group
#> PsyR used df between of: 3
#> PsyR assumed within subject factor(s) are: spacing
#> PsyR used df within of: 2
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality with other tables was not applicable because only one table was supplied.
#> PsyR scaled estimate, SE, and confidence limits by sample SD: 4.625563
```

Or even better, you can do all three at once:


``` r

# enter contrast tables in one long list

psyci(model=mod, contrast_tables = list(btwn_con, con_win, con_int),
      method="ph")
#> Raw contrast effects and confidence intervals (dependent-variable units)
#>
#> $b
#>   contrast   estimate       SE df_error    F_value psyr_p_value       cc
#> 1   12vs34   2.958333 2.115720       12  1.9551390  0.596976752 3.235875
#> 2     1vs2 -14.083333 2.992081       12 22.1546218  0.004609298 3.235875
#> 3     3vs4  -2.500000 2.992081       12  0.6981254  0.871845486 3.235875
#>        lower     upper
#> 1  -3.887873  9.804540
#> 2 -23.765331 -4.401336
#> 3 -12.181998  7.181998
#>
#> Results are averaged over the levels of: spacing
#> PsyR CI method: Scheffe has been applied
#> PsyR p-value method: Scheffe has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: between subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed between subject factor(s) are: group
#> PsyR used df between of: 3
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality: this table is orthogonal to contrast_tables[[2]], contrast_tables[[3]].
#>
#> $w
#>   contrast estimate       SE df_error   F_value psyr_p_value       cc     lower
#> 1   20vs40    -1.75 0.832291       12 4.4210526   0.17815228 2.947652 -4.203304
#> 2   20vs60    -3.00 1.014222       12 8.7493671   0.04920057 2.947652 -5.989572
#> 3     Quad    -0.25 0.458570       12 0.2972136   0.87409756 2.947652 -1.601705
#>        upper
#> 1  0.7033040
#> 2 -0.0104275
#> 3  1.1017046
#>
#> Results are averaged over the levels of: group
#> PsyR CI method: post-hoc within (Hotelling's T-squared) has been applied
#> PsyR p-value method: post-hoc within (Hotelling's T-squared) has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: within subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed within subject factor(s) are: spacing
#> PsyR used df within of: 2
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality: this table is orthogonal to contrast_tables[[1]], contrast_tables[[3]].
#>
#> $bw
#>   group_custom spacing_custom estimate       SE df_error     F_value
#> 1       12vs34         20vs40   -8.000 1.664582       12 23.09774436
#> 2         1vs2         20vs40    5.000 2.354074       12  4.51127820
#> 3         3vs4         20vs40   -3.000 2.354074       12  1.62406015
#> 4       12vs34         20vs60  -19.250 2.028444       12 90.06075949
#> 5         1vs2         20vs60    8.750 2.868652       12  9.30379747
#> 6         3vs4         20vs60   -6.750 2.868652       12  5.53670886
#> 7       12vs34           Quad    1.625 0.917140       12  3.13931889
#> 8         1vs2           Quad    0.625 1.297032       12  0.23219814
#> 9         3vs4           Quad    0.375 1.297032       12  0.08359133
#>   psyr_p_value       cc      lower      upper
#> 1 2.157666e-02 4.191986 -14.977905  -1.022095
#> 2 5.884826e-01 4.191986  -4.868248  14.868248
#> 3 9.297047e-01 4.191986 -12.868248   6.868248
#> 4 8.158899e-05 4.191986 -27.753208 -10.746792
#> 5 2.240525e-01 4.191986  -3.275351  20.775351
#> 6 4.807223e-01 4.191986 -18.775351   5.275351
#> 7 7.546456e-01 4.191986  -2.219638   5.469638
#> 8 9.995511e-01 4.191986  -4.812140   6.062140
#> 9 9.999771e-01 4.191986  -5.062140   5.812140
#>
#> PsyR CI method: post-hoc between x within (Roy's GCR) has been applied
#> PsyR p-value method: post-hoc between x within (Roy's GCR) has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: between x within subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed between subject factor(s) are: group
#> PsyR used df between of: 3
#> PsyR assumed within subject factor(s) are: spacing
#> PsyR used df within of: 2
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality: this table is orthogonal to contrast_tables[[1]], contrast_tables[[2]].
#>
#> Scaled contrast effects and confidence intervals (sample SD units)
#> Scaling divisor (pooled sample SD): 4.625563
#>
#> $b
#>   contrast   estimate        SE df_error    F_value psyr_p_value       cc
#> 1   12vs34  0.6395618 0.4573974       12  1.9551390  0.596976752 3.235875
#> 2     1vs2 -3.0446744 0.6468576       12 22.1546218  0.004609298 3.235875
#> 3     3vs4 -0.5404747 0.6468576       12  0.6981254  0.871845486 3.235875
#>        lower      upper
#> 1 -0.8405188  2.1196424
#> 2 -5.1378245 -0.9515243
#> 3 -2.6336248  1.5526753
#>
#> Results are averaged over the levels of: spacing
#> PsyR CI method: Scheffe has been applied
#> PsyR p-value method: Scheffe has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: between subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed between subject factor(s) are: group
#> PsyR used df between of: 3
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality: this table is orthogonal to contrast_tables[[2]], contrast_tables[[3]].
#> PsyR scaled estimate, SE, and confidence limits by sample SD: 4.625563
#>
#> $w
#>   contrast    estimate        SE df_error   F_value psyr_p_value       cc
#> 1   20vs40 -0.37833232 0.1799329       12 4.4210526   0.17815228 2.947652
#> 2   20vs60 -0.64856969 0.2192645       12 8.7493671   0.04920057 2.947652
#> 3     Quad -0.05404747 0.0991382       12 0.2972136   0.87409756 2.947652
#>        lower        upper
#> 1 -0.9087119  0.152047212
#> 2 -1.2948851 -0.002254321
#> 3 -0.3462724  0.238177416
#>
#> Results are averaged over the levels of: group
#> PsyR CI method: post-hoc within (Hotelling's T-squared) has been applied
#> PsyR p-value method: post-hoc within (Hotelling's T-squared) has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: within subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed within subject factor(s) are: spacing
#> PsyR used df within of: 2
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality: this table is orthogonal to contrast_tables[[1]], contrast_tables[[3]].
#> PsyR scaled estimate, SE, and confidence limits by sample SD: 4.625563
#>
#> $bw
#>   group_custom spacing_custom    estimate        SE df_error     F_value
#> 1       12vs34         20vs40 -1.72951919 0.3598658       12 23.09774436
#> 2         1vs2         20vs40  1.08094949 0.5089271       12  4.51127820
#> 3         3vs4         20vs40 -0.64856969 0.5089271       12  1.62406015
#> 4       12vs34         20vs60 -4.16165554 0.4385290       12 90.06075949
#> 5         1vs2         20vs60  1.89166161 0.6201737       12  9.30379747
#> 6         3vs4         20vs60 -1.45928181 0.6201737       12  5.53670886
#> 7       12vs34           Quad  0.35130858 0.1982764       12  3.13931889
#> 8         1vs2           Quad  0.13511869 0.2804052       12  0.23219814
#> 9         3vs4           Quad  0.08107121 0.2804052       12  0.08359133
#>   psyr_p_value       cc      lower      upper
#> 1 2.157666e-02 4.191986 -3.2380717 -0.2209666
#> 2 5.884826e-01 4.191986 -1.0524660  3.2143650
#> 3 9.297047e-01 4.191986 -2.7819852  1.4848458
#> 4 8.158899e-05 4.191986 -5.9999631 -2.3233480
#> 5 2.240525e-01 4.191986 -0.7080979  4.4914211
#> 6 4.807223e-01 4.191986 -4.0590413  1.1404777
#> 7 7.546456e-01 4.191986 -0.4798634  1.1824806
#> 8 9.995511e-01 4.191986 -1.0403360  1.3105734
#> 9 9.999771e-01 4.191986 -1.0943835  1.2565259
#>
#> PsyR CI method: post-hoc between x within (Roy's GCR) has been applied
#> PsyR p-value method: post-hoc between x within (Roy's GCR) has been applied to `psyr_p_value` using the displayed `F_value`.
#> Family-wise correction assumes current contrasts are: between x within subject contrasts
#> PsyR used an alpha rate of: 0.05
#> PsyR assumed between subject factor(s) are: group
#> PsyR used df between of: 3
#> PsyR assumed within subject factor(s) are: spacing
#> PsyR used df within of: 2
#> PsyR used df error of: 12
#> PsyR found no non-contrast rows to drop.
#> Orthogonality: this table is orthogonal to contrast_tables[[1]], contrast_tables[[2]].
#> PsyR scaled estimate, SE, and confidence limits by sample SD: 4.625563
```
