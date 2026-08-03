# Terraform Install Failure on Linux Mint — Codename Mismatch

## Problem
Terraform's apt repo setup failed because it expects an Ubuntu codename, 
but Linux Mint reports its own codename (e.g. "zena") which apt doesn't recognize.

## Solution
Manually mapped the Mint codename to its underlying Ubuntu base codename 
("zena" -> "noble") when configuring the apt source, so the repository 
URL resolved correctly.

## Lesson
Anything Ubuntu-specific (PPAs, apt repos) on Mint needs the Ubuntu base 
codename, not the Mint release name.
