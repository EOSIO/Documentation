# Account Recovery

This document describes the various account recovery features in EOS.IO.

EOS.IO is designed foremost with real people in mind.  People make mistakes.  Any system designed with people in mind must provide reasonable constructs and processes for minimizing the damage caused by simple human error.

Counter to this principle, many existing blockchains excessively penalize human errors.  In some cases, it results in an inconvenient delay in transaction processing however, in _most_ cases it results in irreversible loss of funds and other material damage that is excessive considering the underlying cause of the problem.  This cannot be a property of any human-centric blockchain going forward.

The account recovery features of EOS.IO bridge the gap between the uncompromising security of inflexible and unapologizing account structure of current blockchains, and the needs of ordinary people who should be able to benefit from these blockchains but could not reasonably and safely do so.

## Terms and Definitions

 * **Account:** the simplest representation of user or contract and is associated with a human readable name eg "Alice"
 * **Authority:** a blueprint of public keys and references to other accounts used to authorize actions on behalf of an account.
 * **Permission:** human readable name that associates an Authority to an Account.  There are 3 special permissions:
   * **owner:** the highest permission for an account.  It constitutes total control and has special rules to protect it.
   * **active:** the highest permission without special rules attached, able to authorize any actions
   * **recovery:** a special permission which cannot authorize any actions except for account recovery
 * **Multi-Sig Authority:** an authority which authorizes actions if-and-only-if it is satisfied by a number of private keys or referenced accounts
   * a basic Multi-Sig Authority can be represented by two quantities M and N, where M is the number of required authorizers and N is the number of potential authorizers
   * EOS.IO supports a more flexible model where each authorizing party (a public key or referenced account) has a weight and the Authority is satisfied when the sum of all authorizing weights exceeds a threshold.  This is a strict superset of the basic multi-sig authority.

## Basic Operational Restrictions

The owner permission is the apex of the permission structure.  As a result, updates to the owner permission are restricted so that interested parties have time to react should an update to the owner permission be initiated by a malicious party.  This update period is 30days during which the pending update is publically reviewable and cancellable by using the proper keys as defined by the owner authority itself.

The active permission is directly underneath the owner permission and carries no such restriction on updates.  Without engaging in a sepcialized recovery process the active permission cannot change the owner permission OR the recovery permission.  Otherwise, the active permission has complete control over the account.

The recovery permission is also directly underneath the owner permission.  The authority associated with it can be changed at any time by initiating an update from the owner permission.  Unlike other permissions, satisfying the recovery permission is not sufficient to update itself.  The recovery permission is also restricted in that it cannot authorize _any_ actions.

Neither the active nor the recovery permissions can be re-parented to anything other than the owner permission.

## Best Practices

EOS.IO does not enforce any restrictions on the Authorities associated with an account at _any_ permission levels, however, best practices detail resonable defaults for some of the special permissions designed to keep the average user safe and secure.

More succintly, the EOS.IO blockchains software has no restrictions however, it is suggested that wallets and other customer facing tools implement these best practices.

### Owner Permission

The owner permission can be used to do anything on an account, like "root" on a unix system or the super-user on windows.  Because of its capacity for bad actors to do bad things, it should rarely be used.  Best practices are to only use the owner permission when specifically setting the Authority for the active permission or when participating in some of the recovery procedures outlined below.

Furthermore, the owner permission should be extremely hard to compromise.  As such, best practices suggest that the authority set on an account's owner permission a multi-sig authority which references a key the user controls and keeps private and any number of trusted 3rd parties.  Weights for this authority should be set such that the users key is _always_ required and additionally some M of N trusted 3rd parties are required as well.  For example, if the user's key has a weight of 4 and there are 3 trusted 3rd party accounts each with a weight of 1, then an authority with a threshold of 6 can only be satisfied by the users key AND 2 of 3 trusted parties.

For new accounts, the account of the creator is recommended as the first trusted 3rd party.  Afterwards, the user may reconfigure that set of accounts to any set of other accounts.  As discussed later in the recovery procedures, this will not afford any of the trusted 3rd parties the ability to hold the owner account hostage.

These best practices do a number of things:

  * Isolate the owner permission and treat it more securely
  * Require compromising some number of trusted 3rd parties AND the a private key the user controls in order to compromise any user's owner permission

### Recovery Permission

The recovery permission is a special permission used during the recovery process.  There are checks and balances to these processes, outlined below, however there are still best practices that should be in place.

It is recommended that the recovery permission be set to the same parties as the owner permission with the N of N relaxed to 1 of N.  This means that any of the user's trusted 3rd parties can help facilitate a recovery action.  As discussed below this does not give these trusted 3rd parties unilateral authority to compromise an account.  It is also worth noting that the recovery permission cannot authorize any non-recovery actions.

## Recovery Procedures

In this section, the procedures available for recovery are described without any context.  Later in the use case studies, the recovery procedures can be matched to scenarios to provide context.

### Recovering Owner from Active

At any time, the active permission may issue a `postrecovery` action to the EOS.IO contract to initiate a recovery process.  This will automatically generate a `passrecovery` action with a delay of 30 days before this action is applied.  During that time, the authorizing parties as well as the desired replacement for the Authority to be associated with the owner permission are publicly available for review.  Additionally, the action is cancellable by issuing a `vetorecovery` action.

If the pending `passrecovery` action is not cancelled, when it is applied the authority associated with the  owner permission will be changed immediately (without an additional 30 day delay a normal owner update must suffer)

### Recovering Owner from idle

If neither the owner or active permission has been used to authorize any action in the past 30 days, the account will be considered idle.  The recovery permission can initiate a recovery process on an idle account by sending a `postrecovery` action to the EOS.IO contract.  This will automatically generate a `passrecovery` action with a delay of 7 days before application.  During that time, authorizing _any_ action with either the owner or active permission will invalidate the recovery.  Additionally, the action is directly cancellable by issuing a properly authorized `vetorecovery` action.

As with recoverying owner from active, when the `passrecovery action is applied it will apply immediately.

## Use Case Studies

### Compromised Active Authority

Assume Alice has set up an account following best practices, and Mallory has somehow compromised or stolen Alice's private key.  The active permission is authorized by this key, and Mallory has just gained the ability to authorize actions as Alice.

If Mallory is detected before issueing an update to the active permissions authority, Alice may use the compromised key to instantly update the active authority to reference a new and secure key.  However, it is more likely that Mallory will issue this update and Alice will no longer have the key(s) associated with the active permission authority.

There is no recovery process necessary in this case, as Alice can contact her trusted 3rd parties who, after verifying Alice is the _real_ Alice will co-sign an update from the owner permission, to replace the compromised key with a new key.

As there is still some damage that can be done in the interim, it is recommened that user's take advantage of the deep permission tree structure available to EOS.IO to reduce the usage of permission levels like active for sensitive actions, such as the transfer of funds, OR to further protect those actions using smart contracts outside the scope of this document.

### Owner Permission Held Hostage

Assume Alice has set up an account following best practices, and Mallory is Alice's trusted 3rd party who is now acting in bad faith.  As Alice's owner permission authority is only satisfied by 2 of 2 consent from Alice _and_ Mallory, Alice has lost the ability to authorize actions from the owner permission.  Unchecked, this could present a denial-of-service for updating the owner permission, the active permission, and the recovery permission authorities.

Alice may use her private key to sign a `postrecovery` action authorized by her active permission and send it to the EOS.IO contract.  The action is formatted such that once applied, Mallory will no longer be part of Alice's owner permission.  This triggers [Recovering Owner from Active](#recovering-owner-from-active).

This action is cancellable by the active permission and the owner permission.  So long as Alice has securely retained her active key which is also referenced by the 2 of 2 multi-sig authority associated with her owner permission, no cancellation action can be signed without her consent.  Furthermore, no action can be signed with the owner permission without her consent.

Given this, Mallory is unable to stop the recovery and unable to issue any actions on Alice's behalf.  After a 30 day delay, Alice's new owner authority will take control and she may use that to instantly re-configure the recovery permission if Mallory is still referenced by it.

This gives Alice a guaranteed path to restitution.  Additionally, it is trivial to see that for 30 days, Mallory was acting in bad faith and if Mallory has made a business of being a trusted 3rd party, her reputation will be materially damaged by such proof.  As a hedge against Alice attempting to maliciously damage Mallory's reputation, Mallory could, at any time, have publically published a signed action which represented the same owner authority update, proving her consent to Alice's wishes.

### Forgotten/Lost Key Recovery

Assume Alice has set up an account following best practices, and has forgotten or otherwise lost the private key associated with her active permission.  As Alice's owner permission authority requires that key as well, Alice has lost the ability to authorize actions with the owner and active permissions even with the cooperation of her trusted 3rd parties.

After 30 days, Alice's account will be considered "idle" even if she has continued to issue actions with permissions which are not active or owner.  Once idle, and one of her trusted 3rd parties can initiate [Recovering Owner from Idle](#recovering-owner-from-idle) on her behalf.  Assuming Alice has more than one trusted 3rd party, she can request any of them cancel such an action so, as long as one trusted 3rd party remains faithful Alice should be able to prevent the abuse of this method to set her owner permission authority to something she would not consent to.

### Compromised Trusted 3rd Party

Assume Alice has set up an account following best practices, and that one or all of her trusted 3rd parties (Bob) has been compromised by Mallory.  Alice's owner permission auhtoriy and recovery authorities now reference an account which can be fraudulently authorized by Mallory insted of Bob.

With only Bob's account, Mallory will be unable to authorize an action using the owner permission on Alice's account; that would require consent from Alice.  Mallory can however, issue a recovery request if Alice is deemed idle however, that attack is easily defeated by Alice signing any action (including a `nonce`) with her active key.  Likewise, any additional trusted 3rd party can cancel that request on Alice's behalf if they are listed in the recovery permission's associated authority.  Bob will need to initiate his own recovery procedure to remove Mallory's influence.  Once complete all authorities which reference Bob's account (as opposed to directly referencing keys Bob holds) will be automatically updated to reflect the new authorities on Bob's account.

In this scenario, Alice or another trusted 3rd party may play a minor role but Mallory is unable to authorize actions on behalf of Alice and Alice will not necessarily need to take any action to keep up-to-date with Bob's latest and most secure keys.

## Conclusion

By deploying best practices when setting up permissions on new accounts, users can hold a reasonable degree of faith that if their account or related accounts are compromised that there exists mitigations to prevent extreme and/or permanent loss of resources.

When combined with the deep permission capabilities of EOS.IO to provide configurable levels of authentication on every aspect of an account, these recovery procedures present one of, if not the most human error tolerant blockchain authorization systems available. While maintaining security strong enough to guard extremely valuable assets.