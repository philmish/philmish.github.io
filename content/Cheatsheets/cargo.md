---
title: "Cargo"
date: 2023-07-01T20:30:38+02:00
draft: false
tags: ["rust", "cargo"]
---

Cargo is a package manager for Rust. It comes with most Rust installations. 
It manages downloading packages and dependecies, compiles your packages and can be used to upload it to package registries.

## Basics

To create a new package and create the related directory use:
{{% command %}}cargo new directory_name{{% /command %}}


To build your binary inside your package's /target/debug/<package_name> directory run:
{{% command %}}cargo build{{% /command %}}

If you want to build your binary with optimizations for release run:
{{% command %}}cargo build --release{{% /command %}}


To compile and run your binary use:
{{% command %}}cargo run{{% /command %}}







