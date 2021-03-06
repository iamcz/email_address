# Email Address

[![Gem Version](https://badge.fury.io/rb/email_address.svg)](http://rubygems.org/gems/email_address)
[![Build Status](https://travis-ci.org/afair/email_address.svg?branch=v0.1)](https://travis-ci.org/afair/email_address)
[![Code Climate](https://codeclimate.com/github/afair/email_address/badges/gpa.svg)](https://codeclimate.com/github/afair/email_address)

The `email_address` gem provides a ruby language library for working
with email addresses.

By default, it validates against conventional usage,
the format preferred for user email addresses.
It can be configured to validate against RFC "Standard" formats,
common email service provider formats, and perform DNS validation.

Using `email_address` to validate user email addresses results in
fewer "false positives" due to typing errors and gibberish data.
It validates syntax more strictly for popular email providers,
and can deal with gmail's "optional dots" in addresses.

It provides Active Record (Rails) extensions, including an
address validator and attributes API custom datatypes.

*Note:* Version 0.1.0 contains significant API and internal changes over the 0.0.3
version. If you have been using the 0.0.x series of the gem, you may
want to continue using with your current version.

Requires Ruby 2.0 or later.

## Quick Start

To quickly validate email addresses, use the valid? and error helpers.
`valid?` returns a boolean, and `error` returns nil if valid, otherwise
a basic error message.

    EmailAddress.valid? "allen@google.com" #=> true
    EmailAddress.error "allen@bad-d0main.com" #=> "Invalid Host/Domain Name"

`EmailAddress` deeply validates your email addresses. It checks:

* Host name format and DNS setup
* Mailbox format according to "conventional" form. This matches most used user
  email accounts, but is a subset of the RFC specification.

It does not check:

* The mail server is configured to accept connections
* The mailbox is valid and accepts email.

By default, MX records are required in DNS. MX or "mail exchanger" records
tell where to deliver email for the domain. Many domains run their
website on one provider (ISP, Heroku, etc.), and email on a different
provider (such as Google Apps).  Note that `example.com`, while
a valid domain name, does not have MX records.

    EmailAddress.valid? "allen@example.com" #=> false
    EmailAddress.valid? "allen@example.com", dns_lookup: :off #=> true

Most mail servers do not yet support Unicode mailboxes, so the default here is ASCII.

    EmailAddress.error "Pelé@google.com" #=> "Invalid Recipient/Mailbox"
    EmailAddress.valid? "Pelé@google.com", local_encoding: :unicode #=> true


## Background

The email address specification is complex and often not what you want
when working with personal email addresses in applications. This library
introduces terms to distinguish types of email addresses.

* *Normal* - The edited form of any input email address. Typically, it
  is lower-cased and minor "fixes" can be performed, depending on the
  configurations and email address provider.

    CKENT@DAILYPLANET.NEWS => ckent@dailyplanet.news

* *Conventional* - Most personal account addresses are in this basic
  format, one or more "words" separated by a single simple punctuation
  character. It consists of a mailbox (user name or role account) and
  an optional address "tag" assigned by the user.

    miles.o'brien@ncc-1701-d.ufp

* *Relaxed* - A less strict form of Conventional, same character set,
  must begin and end with an alpha-numeric character, but order within
  is not enforced.

    aasdf-34-.z@example.com

* *Standard* - The RFC-Compliant syntax of an email address. This is
  useful when working with software-generated addresses or handling
  existing email addresses, but otherwise not useful for personal
  addresses.

    madness!."()<>[]:,;@\\\"!#$%&'*+-/=?^_`{}| ~.a(comment )"@example.org

* *Canonical* - An unique account address, lower-cased, without the
  tag, and with irrelevant characters stripped.

    clark.kent+scoops@gmail.com => clarkkent@gmail.com

* *Reference* - The MD5 of the Canonical format, used to share account
  references without exposing the private email address directly.

    Clark.Kent+scoops@gmail.com => c5be3597c391169a5ad2870f9ca51901

* *Redacted* - A form of the email address where it is replaced by
  a SHA1-based version to remove the original address from the
  database, or to store the address privately, yet still keep it
  accessible at query time by converting the queried address to
  the redacted form.

    Clark.Kent+scoops@gmail.com => {bea3f3560a757f8142d38d212a931237b218eb5e}@gmail.com

* *Munged* - An obfuscated version of the email address suitable for
  publishing on the internet, where email address harvesting
  could occur.

    Clark.Kent+scoops@gmail.com => cl\*\*\*\*\*@gm\*\*\*\*\*

Other terms:

* *Local* - The left-hand side of the "@", representing the user,
  mailbox, or role, and an optional "tag".

    mailbox+tag@example.com;   Local part: mailbox+tag

* *Mailbox* - The destination user account or role account.
* *Tag* - A parameter added after the mailbox, usually after the
  "+" symbol, set by the user for mail filtering and sub-accounts.
  Not all mail systems support this.
* *Host* (sometimes called *Domain*) - The right-hand side of the "@"
  indicating the domain or host name server to delivery the email.
  If missing, "localhost" is assumed, or if not a fully-qualified
  domain name, it assumed another computer on the same network, but
  this is increasingly rare.
* *Provider* - The Email Service Provider (ESP) providing the email
  service. Each provider may have its own email address validation
  and canonicalization rules.
* *Punycode* - A host name with Unicode characters (International
  Domain Name or IDN) needs conversion to this ASCII-encoded format
  for DNS lookup.

    "HIRO@こんにちは世界.com" => "hiro@xn--28j2a3ar1pp75ovm7c.com"

Wikipedia has a great article on
[Email Addresses](https://en.wikipedia.org/wiki/Email_address),
much more readable than the section within
[RFC 5322](https://tools.ietf.org/html/rfc5322#section-3.4)

## Avoiding the Bad Parts of RFC Specification

Following the RFC specification sounds like a good idea, until you
learn about all the madness contained therein. This library can
validate the RFC syntax, but this is never useful, especially when
validating user email address submissions. By default, it validates
to the *conventional* format.

Here are a few parts of the RFC specification you should avoid:

* Case-sensitive local parts: `First.Last@example.com`
* Spaces and Special Characters: `"():;<>@[\\]`
* Quoting and Escaping Requirements: `"first \"nickname\" last"@example.com`
* Comment Parts: `(comment)mailbox@example.com`
* IP and IPv6 addresses as hosts: `mailbox@[127.0.0.1]`
* Non-ASCII (7-bit) characters in the local part: `Pelé@example.com`
* Validation by voodoo regular expressions
* Gmail allows ".." in addresses since they are not meaningful, but
  the standard does not.

## Installation With Rails or Bundler

If you are using Rails or a project with Bundler, add this line to your application's Gemfile:

    gem 'email_address'

And then execute:

    $ bundle

## Installation Without Bundler

If you are not using Bundler, you need to install the gem yourself.

    $ gem install email_address

Require the gem inside your script.

    require 'rubygems'
    require 'email_address'

## Usage

Use `EmailAddress` to do transformations and validations. You can also
instantiate an object to inspect the address.

These top-level helpers return edited email addresses and validation
check.

    address = "Clark.Kent+scoops@gmail.com"
    EmailAddress.valid?(address)    #=> true
    EmailAddress.normal(address)    #=> "clark.kent+scoops@gmail.com"
    EmailAddress.canonical(address) #=> "clarkkent@gmail.com"
    EmailAddress.reference(address) #=> "c5be3597c391169a5ad2870f9ca51901"
    EmailAddress.redact(address)    #=> "{bea3f3560a757f8142d38d212a931237b218eb5e}@gmail.com"
    EmailAddress.munge(address)     #=> "cl*****@gm*****"
    EmailAddress.matches?(address, 'google') #=> 'google' (true)
    EmailAddress.error("#bad@example.com") #=> "Invalid Mailbox"

Or you can create an instance of the email address to work with it.

    email = EmailAddress.new(address) #=> #<EmailAddress::Address:0x007fe6ee150540 ...>
    email.normal        #=> "clark.kent+scoops@gmail.com"
    email.canonical     #=> "clarkkent@gmail.com"
    email.original      #=> "Clark.Kent+scoops@gmail.com"
    email.valid?        #=> true

Here are some other methods that are available.

    email.redact        #=> "{bea3f3560a757f8142d38d212a931237b218eb5e}@gmail.com"
    email.sha1          #=> "bea3f3560a757f8142d38d212a931237b218eb5e"
    email.md5           #=> "c5be3597c391169a5ad2870f9ca51901"
    email.host_name     #=> "gmail.com"
    email.provider      #=> :google
    email.mailbox       #=> "clark.kent"
    email.tag           #=> "scoops"

    email.host.exchanger.first[:ip] #=> "2a00:1450:400b:c02::1a"
    email.host.txt_hash #=> {:v=>"spf1", :redirect=>"\_spf.google.com"}

    EmailAddress.normal("HIRO@こんにちは世界.com")
                        #=> "hiro@xn--28j2a3ar1pp75ovm7c.com"
    EmailAddress.normal("hiro@xn--28j2a3ar1pp75ovm7c.com", host_encoding: :unicode)
                        #=> "hiro@こんにちは世界.com"

#### Rails Validator

For Rails' ActiveRecord classes, EmailAddress provides an ActiveRecordValidator.
Specify your email address attributes with `field: :user_email`, or
`fields: [:email1, :email2]`. If neither is given, it assumes to use the
`email` or `email_address` attribute.

    class User < ActiveRecord::Base
      validates_with EmailAddress::ActiveRecordValidator, field: :email
    end

#### Rails Email Address Type Attribute

Initial support is provided for Active Record 5.0 attributes API.

First, you need to register the type in
`config/initializers/email_address.rb` along with any global
configurations you want.

    ActiveRecord::Type.register(:email_address, EmailAddress::EmailAddressType)
    ActiveRecord::Type.register(:canonical_email_address,
                                EmailAddress::CanonicalEmailAddressType)

Assume the Users table contains the columns "email" and "canonical_email".
We want to normalize the address in "email" and store the canonical/unique
version in "canonical_email". This code will set the canonical_email when
the email attribute is assigned. With the canonical_email column,
we can look up the User, even it the given email address didn't exactly
match the registered version.

    class User < ApplicationRecord
      attribute :email, :email_address
      attribute :canonical_email, :canonical_email_address

      validates_with EmailAddress::ActiveRecordValidator,
                     fields: %i(email canonical_email)

      def email=(email_address)
        self[:canonical_email] = email_address
        self[:email] = email_address
      end

      def self.find_by_email(email)
        user   = self.find_by(email: EmailAddress.normal(email))
        user ||= self.find_by(canonical_email: EmailAddress.canonical(email))
        user ||= self.find_by(canonical_email: EmailAddress.redacted(email))
        user
      end

      def redact!
        self[:canonical_email] = EmailAddress.redact(self.canonical_email)
        self[:email]           = self[:canonical_email]
      end
    end

Here is how the User model works:

    user = User.create(email:"Pat.Smith+registrations@gmail.com")
    user.email           #=> "pat.smith+registrations@gmail.com"
    user.canonical_email #=> "patsmith@gmail.com"
    User.find_by_email("PAT.SMITH@GMAIL.COM")
                    #=> #<User email="pat.smith+registrations@gmail.com">


The `find_by_email` method looks up a given email address by the
normalized form (lower case), then by the canonical form, then finally
by the redacted form.

#### Validation

The only true validation is to send a message to the email address and
have the user (or process) verify it has been received. Syntax checks
help prevent erroneous input. Even sent messages can be silently
dropped, or bounced back after acceptance.  Conditions such as a
"Mailbox Full" can mean the email address is known, but abandoned.

There are different levels of validations you can perform. By default, it will
validate to the "Provider" (if known), or "Conventional" format defined as the
"default" provider. You may pass a a list of parameters to select
which syntax and network validations to perform.

#### Comparison

You can compare email addresses:

    e1 = EmailAddress.new("Clark.Kent@Gmail.com")
    e2 = EmailAddress.new("clark.kent+Superman@Gmail.com")
    e3 = EmailAddress.new(e2.redact)
    e1.to_s           #=> "clark.kent@gmail.com"
    e2.to_s           #=> "clark.kent+superman@gmail.com"
    e3.to_s           #=> "{bea3f3560a757f8142d38d212a931237b218eb5e}@gmail.com"

    e1 == e2          #=> false (Matches by normalized address)
    e1.same_as?(e2)   #=> true  (Matches as canonical address)
    e1.same_as?(e3)   #=> true  (Matches as redacted address)
    e1 < e2           #=> true  (Compares using normalized address)

#### Matching

Matching addresses by simple patterns:

   * Top-Level-Domain:         .org
   * Domain Name:              example.com
   * Registration Name:        hotmail.   (matches any TLD)
   * Domain Glob:              *.exampl?.com
   * Provider Name:            google
   * Mailbox Name or Glob:     user00*@
   * Address or Glob:          postmaster@domain*.com
   * Provider or Registration: msn

Usage:

    e = EmailAddress.new("Clark.Kent@Gmail.com")
    e.matches?("gmail.com") #=> true
    e.matches?("google")    #=> true
    e.matches?(".org")      #=> false
    e.matches?("g*com")     #=> true
    e.matches?("gmail.")    #=> true
    e.matches?("*kent*@")   #=> true

### Configuration

You can pass an options hash on the `.new()` and helper class methods to
control how the library treats that address. These can also be
configured during initialization by provider and default (see below).

    EmailAddress.new("clark.kent@gmail.com",
                     dns_lookup::off, host_encoding: :unicode)

Globally, you can change and query configuration options:

    EmailAddress::Config.setting(:dns_lookup, :mx)
    EmailAddress::Config.setting(:dns_lookup) #=> :mx

Or set multiple settings at once:

    EmailAddress::Config.configure(local_downcase:false, dns_lookup: :off)

You can add special rules by domain or provider. It takes the options
above and adds the :domain_match and :exchanger_match rules.

    EmailAddress.define_provider('google',
      domain_match:      %w(gmail.com googlemail.com),
      exchanger_match:   %w(google.com), # Requires dns_lookup==:mx
      local_size:        5..64,
      mailbox_canonical: ->(m) {m.gsub('.','')})

The library ships with the most common set of provider rules. It is not meant
to house a database of all providers, but a separate `email_address-providers`
gem may be created to hold this data for those who need more complete rules.

Personal and Corporate email systems are not intended for either solution.
Any of these email systems may be configured locally.

Pre-configured email address providers include: Google (gmail), AOL, MSN
(hotmail, live, outlook), and Yahoo. Any address not matching one of
those patterns use the "default" provider rule set. Exchanger matches
matches against the Mail Exchanger (SMTP receivers) hosts defined in
DNS. If you specify an exchanger pattern, but requires a DNS MX lookup.

For Rails application, create an initializer file with your default
configuration options:

    # ./config/initializers/email_address.rb
    EmailAddress::Config.setting( local_format: :relaxed )
    EmailAddress::Config.provider(:github,
           host_match: %w(github.com), local_format: :standard)

#### Override Error Messaegs

You can override the default error messages as follows:

    EmailAddress::Config.error_messages(
      invalid_address:    "Invalid Email Address",
      invalid_mailbox:    "Invalid Recipient/Mailbox",
      invalid_host:       "Invalid Host/Domain Name",
      exceeds_size:       "Address too long",
      not_allowed:        "Address is not allowed",
      incomplete_domain:  "Domain name is incomplete")

Full translation support would be ideal though.


### Available Configuration Settings

* dns_lookup: Enables DNS lookup for validation by
    * :mx       - DNS MX Record lookup
    * :a        - DNS A Record lookup (as some domains don't specify an MX incorrectly)
    * :off      - Do not perform DNS lookup (Test mode, network unavailable)

* sha1_secret -
  This application-level secret is appended to the email_address to compute
  the SHA1 Digest, making it unique to your application so it can't easily be
  discovered by comparing against a known list of email/sha1 pairs.

* munge_string - "*****", the string to replace into munged addresses.

For local part configuration:

* local_downcase: true.
  Downcase the local part. You probably want this for uniqueness.
  RFC says local part is case insensitive, that's a bad part.

* local_fix:  true.
  Make simple fixes when available, remove spaces, condense multiple punctuations

* local_encoding:     :ascii, :unicode,
  Enable Unicode in local part. Most mail systems do not yet support this.
  You probably want to stay with ASCII for now.

* local_parse:        nil, ->(local) { [mailbox, tag, comment] }
  Specify an optional lambda/Proc to parse the local part. It should return an
  array (tuple) of mailbox, tag, and comment.

* local_format:
    * :conventional - word ( puncuation{1} word )*
    * :relaxed      - alphanum ( allowed_characters)* alphanum
    * :standard     - RFC Compliant email addresses (anything goes!)

* local_size:         1..64,
  A Range specifying the allowed size for mailbox + tags + comment

* tag_separator:      nil, character (+)
  Nil, or a character used to split the tag from the mailbox

For the mailbox (AKA account, role), without the tag
* mailbox_size:       1..64
  A Range specifying the allowed size for mailbox

* mailbox_canonical:  nil, ->(mailbox) { mailbox }
  An optional lambda/Proc taking a mailbox name, returning a canonical
  version of it. (E.G.: gmail removes '.' characters)

* mailbox_validator:  nil, ->(mailbox) { true }
  An optional lambda/Proc taking a mailbox name, returning true or false.

* host_encoding:      :punycode,  :unicode,
  How to treat International Domain Names (IDN). Note that most mail and
  DNS systems do not support unicode, so punycode needs to be passed.
  :punycode           Convert Unicode names to punycode representation
  :unicode            Keep Unicode names as is.

* host_validation:
  :mx                 Ensure host is configured with DNS MX records
  :a                  Ensure host is known to DNS (A Record)
  :syntax             Validate by syntax only, no Network verification
  :connect            Attempt host connection (not implemented, BAD!)

* host_size:          1..253,
  A range specifying the size limit of the host part,

* host_allow_ip:      false,
  Allow IP address format in host: [127.0.0.1], [IPv6:::1]

* address_validation: :parts, :smtp, ->(address) { true }
  Address validation policy
  :parts              Validate local and host.
  :smtp               Validate via SMTP (not implemented, BAD!)
  A lambda/Proc taking the address string, returning true or false

* address_size:       3..254,
  A range specifying the size limit of the complete address

* address_local:      false,
  Allow localhost, no domain, or local subdomains.

For provider rules to match to domain names and Exchanger hosts
The value is an array of match tokens.
* host_match:         %w(.org example.com hotmail. user*@ sub.*.com)
* exchanger_match:    %w(google.com 127.0.0.1 10.9.8.0/24 ::1/64)

## Notes

#### Internationalization

The industry is moving to support Unicode characters in the local part
of the email address. Currently, SMTP supports only 7-bit ASCII, but a
new `SMTPUTF8` standard is available, but not yet widely implemented.
To work properly, global Email systems must be converted to UTF-8
encoded databases and upgraded to the new email standards.

The problem with i18n email addresses is that support outside of the
given locale becomes hard to enter addresses on keyboards for another
locale. Because of this, internationalized local parts are not yet
supported by default. They are more likely to be erroneous.

Proper personal identity can still be provided using
[MIME Encoded-Words](http://en.wikipedia.org/wiki/MIME#Encoded-Word)
in Email headers.


#### Email Addresses as Sensitive Data

Like Social Security and Credit Card Numbers, email addresses are
becoming more important as a personal identifier on the internet.
Increasingly, we should treat email addresses as sensitive data. If your
site/database becomes compromised by hackers, these email addresses can
be stolen and used to spam your users and to try to gain access to their
accounts. You should not be storing passwords in plain text; perhaps you
don't need to store email addresses un-encoded either.

Consider this: upon registration, store the redacted email address for
the user, and of course, the salted, encrypted password.
When the user logs in, compute the redacted email address from
the user-supplied one and look up the record. Store the original address
in the session for the user, which goes away when the user logs out.

Sometimes, users demand you strike their information from the database.
Instead of deleting their account, you can "redact" their email
address, retaining the state of the account to prevent future
access. Given the original email address again, the redacted account can
be identified if necessary.

Because of these use cases, the **redact** method on the email address
instance has been provided.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

#### Project

This project lives at [https://github.com/afair/email_address/](https://github.com/afair/email_address/)

#### Authors

* [Allen Fair](https://github.com/afair) ([@allenfair](https://twitter.com/allenfair)):
  I've worked with email-based applications and email addresses since 1999.
