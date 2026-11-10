# From SMS to the operator panel

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24121806.svg)](https://doi.org/10.5281/zenodo.24121806)

**Threat intelligence case study** · 8.8 Unreal Chile  
**Authors:** Patricio Moraga · Anthony de la Paz  
**TLP:WHITE** (this text) · **Live locators withheld (TLP:AMBER)**

Original research into a **smishing** campaign that does not impersonate a bank. An SMS leads to a fake casino; the fraud continues in the operator chat. This report tells the full hunt and does not publish domains, IPs, shorteners, bots, or passwords: the threat is assessed as still active. Classic defanging (`example[.]com`) is not used, because it still identifies the actor.

## Read the report

1. [Executive summary](docs/01-executive-summary.md)
2. [This is not bank smishing](docs/02-not-bank-smishing.md)
3. [The hunt](docs/03-the-hunt.md)
4. [Attack chain](docs/04-attack-chain.md)
5. [What victims said](docs/05-victim-narratives.md)
6. [ATT&CK, indicators, limits](docs/06-attack-indicators-limits.md)
7. [Recommendations and ethics](docs/07-recommendations-ethics.md)

## Handling

This document is published openly so that authorities, defenders, and the public can review the case. Live infrastructure identifiers are deliberately kept out of this public tree while the campaign is active; they can be made available to competent authorities on request. Do not add live domains, addresses, passwords, or session tokens to this repository.

## Citation

Moraga, P., & de la Paz, A. (2026). *From SMS to the operator panel: hunting a live fake-casino smishing campaign.* Zenodo. https://doi.org/10.5281/zenodo.24121806 (also presented at 8.8 Unreal Chile). See [CITATION.cff](CITATION.cff).

Text is licensed under [CC BY 4.0](LICENSE).
