```rust
use earth::life::person::{ Person, PersonBodyParts as BodyParts };
use kigurumi::Kigurumi;

fn main() -> anyhow::Result<()> {
    let me = Person::me();
    let room = me.room()?;
    let closet = me.closet()?;

    me.wear_cloth(closet.get("skin suit", &me.thinking())?.into());
    me.wear_cloth(closet.get("costume", &me.thinking())?.into());

    /// ```
    /// Kigurumi::new(
    ///     "Silence Suzuka",
    ///     "Umamusume: Pretty Derby",
    ///     KigurumiMaker::StudioRonMaca,
    ///     &me.bank_account
    /// );
    /// ```
    let silence_suzuka: Kigurumi = room.take("kigurumi", &me.thinking())?.into();
    me.put(BodyParts::Head, silence_suzuka);

    Ok(())
}
