---
layout: post
title:  "How to Add a Rust Package to Nix"
date:   2020-06-23 -0700
categories: nix rust
---

This is a quick post on how to contribute a Rust binary to nixpkgs. For more
info, please check out the [official doc](https://nixos.org/nixpkgs/manual/#rust).

First of all, if you've never added any packags to nixpkgs, you need to add
yourself as a maintainer to the `maintainers/maintainer-list.nix` file. Try
keeping the names in sorted order.

Tools are under `pkgs/tools`. Pick a category that is relevant and add a
subdirectory for the Rust package you want to add. For example: if you want to
add the `fastmod` tool, you can add it under `pkgs/tools/text/fastmod`.

Then you need to add a `default.nix` file under the subdirectory you just
created.

Here's the content of the `default.nix` file:

{% highlight rust %}

{ stdenv
, fetchFromGitHub
, rustPlatform
, Security
}:

rustPlatform.buildRustPackage rec {
  pname = "fastmod";
  version = "0.4.0";

  src = fetchFromGitHub {
    owner = "facebookincubator";
    repo = pname;
    rev = "v${version}";
    sha256 = "0089a17h0wgan3fs6x1la35lzjs1pib7p81wqkh3zcwvx8ffa8z8";
  };

  cargoSha256 = "02nkxjwfiljndmi0pv98chfsw9vmjzgmp5r14mchpayp4943qk9m";

  buildInputs = stdenv.lib.optional stdenv.isDarwin Security;

  meta = with stdenv.lib; {
    description = "A utility that makes sweeping changes to large, shared code bases";
    homepage = "https://github.com/facebookincubator/fastmod";
    license = licenses.asl20;
    maintainers = with maintainers; [ jduan ];
    platforms = platforms.all;
  };
}

{% endhighlight %}

A few things worth noting:

* The `fetchFromGitHub` function fetches a release of the package from github.
    The "rev" is a release version.
* Unlike the `rev`, the `version` can only contain digits and dots. You want to
    use `0.4.0` instead of `v0.4.0`.
* For `sha256` and `cargoSha256`, you can put some random text there and wait
    for nix-build to tell you the correct SHAs.

Finally, add your package to `pkgs/top-level/all-packages.nix`.

To test it, you can run the nix-build from the root of the nixpkgs

    repo:nix-build -A fastmod default.nix

That's it! You should see a subdirectory under `/nix/store` and you can verify
the binary by running it.

Now, go to github, open a PR, and have it merged!
